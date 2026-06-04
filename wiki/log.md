# Log

## [2026-04-10] setup | Wiki Initialized
Created directory structure (raw/, wiki/, wiki/sources/, wiki/entities/, wiki/concepts/, wiki/analyses/), CLAUDE.md schema, index.md, and log.md.

## [2026-04-10] ingest | LLM Wiki Idea File
Founding source ingested. Pages created: Source---LLM-Wiki-Idea-File, LLM-Wiki-Pattern, RAG, Memex, Index-Based-Navigation, Vannevar-Bush, Obsidian, qmd. Total: 1 source page, 4 concept pages, 3 entity pages.

## [2026-04-10] ingest | Entra ID OAuth Reference
Source: rajvermacas/concepts — entra-id-oauth-reference.md. Pages created: Source---Entra-ID-OAuth-Reference, Microsoft-Entra-ID, OAuth-2.0-Authorization-Code-Flow, JWT, Service-Principal, App-Registration, JWKS. Total: 1 source page, 5 concept pages, 1 entity page.

## [2026-04-10] ingest | Entra ID Audience Scopes Deep Dive
Source: Socratic Q&A conversation — entra-id-audience-scopes-deep-dive.md. All concepts illustrated using crick-info-buzz app.
Pages created: Source---Entra-ID-Audience-Scopes-Deep-Dive, Scope, PKCE, Public-vs-Confidential-Client, Redirect-URI, MSAL.
Pages updated: App-Registration (added two-registration pattern, public/confidential table), JWT (added aud derivation section).
Total: 1 source page, 4 new concept pages, 1 new entity page, 2 updated concept pages.

## [2026-04-11] lint | 3 contradictions fixed, 4 broken links resolved
Scanned all 26 wiki pages.
Contradictions fixed: (1) OAuth-2.0-Authorization-Code-Flow.md now documents both Pattern 1 (SPA) and Pattern 2 (BFF) with separate step tables, replacing the BFF-only view that was presented as universal. (2) PKCE.md now clarifies that code_verifier is sent by MSAL in Pattern 1 vs forwarded via backend in Pattern 2.
Broken links fixed: (3) Backslash typo in App-Registration.md line 92 (`Public-vs-Confidential-Client\|` → `Public-vs-Confidential-Client|`). (4) Three unresolvable wikilinks in Obsidian.md (Obsidian-Web-Clipper, Dataview, Marp) converted to plain text — these are Obsidian tooling features unlikely to grow into wiki pages.
No orphan pages found. All pages have ≥2 inbound links.
Open questions logged: refresh token renewal in BFF, Conditional Access concept page, token confusion attacks, admin vs user consent mechanics.

## [2026-04-11] ingest | Entra ID App Roles, BFF Pattern, JWT Signing
Source: Socratic session — entra-id-app-roles-bff-jwt-signing.md. Covered all App Registration property sections, App Roles end-to-end mechanics, tenant/subscription cardinality, home tenant definition, SPA vs BFF OAuth patterns, XSS attack vectors (fetch monkey-patching), BFF full cookie traffic path, and RS256 JWT signing mechanics.
Pages created: Source---Entra-ID-App-Roles-BFF-JWT-Signing, App-Roles, BFF-Pattern, XSS, JWT-Signature-Verification.
Pages updated: App-Registration (full property sections, cardinality table), Service-Principal (home vs guest SPN, App Role assignment layer), JWT (roles claim, corrected signing mental model).
Total: 1 raw source, 1 source page, 4 new concept pages, 3 updated concept pages.

## [2026-04-12] ingest | Microsoft Learn Entra ID App Roles Fact Check
Source: Microsoft Learn documentation captured in raw/microsoft-learn-entra-id-app-roles-fact-check-2026-04-12.md.
Pages created: Source---Microsoft-Learn-Entra-ID-App-Roles-Fact-Check.
Pages updated: App-Roles (assignment-required nuance, token claim edge cases), Source---Entra-ID-App-Roles-BFF-JWT-Signing (fact-check correction callout), index.md.

## [2026-04-12] lint | Entra ID app-role fact check and graph scan
Fact-checked the app-role material against current Microsoft Learn docs and corrected the earlier overstatement that Entra ID always issues a token for unassigned principals.
Link graph scan after patching found 4 unresolved wikilinks and fixed them immediately: 3 legacy tool-name links in Source---LLM-Wiki-Idea-File and 1 literal syntax example in Obsidian. No orphan pages remain; the new source page is indexed and cross-linked. Remaining open question: whether to ingest separate Microsoft docs pages for broader Entra topics beyond app roles.

## [2026-04-12] ingest | Microsoft Learn Conditional Access Overview
Source: Microsoft Learn documentation captured in raw/microsoft-learn-conditional-access-overview-2026-04-12.md.
Pages created: Source---Microsoft-Learn-Conditional-Access-Overview, Conditional-Access.
Pages updated: Microsoft-Entra-ID (added Conditional Access control layer), Service-Principal (linked Conditional Access references), index.md.

## [2026-04-12] ingest | Microsoft Learn Permissions and Consent Overview
Source: Microsoft Learn documentation captured in raw/microsoft-learn-permissions-consent-overview-2026-04-12.md.
Pages created: Source---Microsoft-Learn-Permissions-and-Consent-Overview, Consent.
Pages updated: Scope (explicit consent linkage), Microsoft-Entra-ID (consent control layer), index.md.

## [2026-04-14] ingest | Understanding CSR and CER
Source: Claude conversation — raw/Understanding CSR and CER.md. Covered TLS/SSL certificate lifecycle (private key → CSR → CA signing → certificate), the signing-vs-encrypting distinction, signature verification mechanics, file format differences, and practical HashiCorp Vault self-signed CA setup.
Pages created: Source---Understanding-CSR-and-CER, PKI, TLS, CSR, X509-Certificate, Certificate-Authority, Self-Signed-Certificate, PEM-and-DER-Formats, HashiCorp-Vault.
Total: 1 source page, 7 new concept pages, 1 new entity page.

## [2026-04-13] ingest | Auth Flows Taxonomy, Delegated Permissions, OID/Sub Session
Source: Cowork session — session-auth-flows-delegated-oid-sub-2026-04-13.md.
Key findings: (1) `scp` is absent in Client Credentials tokens — confirmed against Microsoft official docs. (2) Scope/App Roles are not mutually exclusive in user tokens. (3) "Delegated" on resource server = user-must-be-present declaration to Entra ID; says nothing about grant flow mechanics. (4) In BFF pattern, Entra ID is off the hot path for all subsequent calls — only re-enters for token refresh (~1hr) and CAE. (5) Delegated vs Application core distinction is user presence, not consent rules. (6) `oid` is tenant-wide stable; `sub` is per-app scoped.
Pages created: Source---Auth-Flows-Delegated-OID-Sub-Session, Delegated-vs-Application-Permissions, OID-and-Sub-Claims, Auth-Flows-Taxonomy (analysis).
Pages updated: BFF-Pattern (subsequent calls section + corrected ASCII diagram), index.md.
Total: 1 raw source, 1 source page, 2 new concept pages, 1 new analysis page, 1 updated concept page.

## [2026-04-14] lint | Lint fixes — cross-links, new concept pages, decision guide
Applied all 5 recommendations from the 2026-04-14 health check:
1. Added [[Auth-Flows-Taxonomy]] cross-links to Delegated-vs-Application-Permissions, Scope, and App-Roles.
2. Added [[HashiCorp-Vault]] cross-link to TLS for the mTLS scenario.
3. Created Token-Expiry-and-Refresh concept page (access/refresh lifetimes, rotation, CAE).
4. Created Multi-Tenant-and-Guest-SPN concept page (guest SPN, cross-tenant consent, CA across tenants).
5. Created Auth-Decision-Guide analysis page (SPA vs BFF, App Roles vs Scopes, XSS, multi-tenant, oid vs sub).
Pages created: Token-Expiry-and-Refresh, Multi-Tenant-and-Guest-SPN, Auth-Decision-Guide.
Pages updated: Delegated-vs-Application-Permissions, Scope, App-Roles, TLS, index.md.

## [2026-04-22] ingest | AKS Workload Identity — AZURE_FEDERATED_TOKEN_FILE
Source: Cowork conversation — raw/aks-workload-identity-federated-token-2026-04-22.md. Covered what AZURE_FEDERATED_TOKEN_FILE is in AKS (env var injected by Workload Identity webhook pointing to a kubelet-managed projected SA token), who creates it (Mutating Admission Webhook for the env var + volume definition; kubelet for the actual token file), the full federated credential exchange flow (K8s JWT → Entra ID → Azure access token), and prerequisites (OIDC issuer, workload-identity flag, SA annotation, pod label, federated credential record).
Pages created: Source---AKS-Workload-Identity-Federated-Token, AKS (entity), Azure-Workload-Identity, Mutating-Admission-Webhook, Federated-Credentials, Projected-Service-Account-Token.
Pages updated: Microsoft-Entra-ID (added Workload Identity section), index.md.
Total: 1 raw source, 1 source page, 1 new entity page, 4 new concept pages, 1 updated entity page.

## [2026-04-22] lint | Post-ingest health check
Scanned all 52 wiki pages (up from 45).
Broken links: 1 found and fixed — Source---AKS-Workload-Identity-Federated-Token had a raw file path as a wikilink `[[raw/aks-...]]`; converted to plain text reference since raw files are not wiki pages.
Orphan pages: 0 — all 7 new pages are well-linked (8–12 inbound links each).
Contradictions: none found. The new Workload Identity material is additive to existing Entra ID and JWT concepts.
Cross-link improvements: (1) JWT.md updated with new section distinguishing Entra-issued vs K8s-issued JWTs and linking to Projected-Service-Account-Token and Federated-Credentials. (2) Service-Principal.md updated with Workload Identity context section linking AKS, Federated-Credentials, and Delegated-vs-Application-Permissions.
Open questions to investigate: (1) How does the AKS OIDC issuer rotate its signing keys, and how does Entra ID handle JWKS cache invalidation? (2) What happens during pod identity in multi-cluster setups — can one Federated Credential trust multiple cluster OIDC issuers? (3) How does Workload Identity interact with Conditional Access policies applied to the managed identity?
