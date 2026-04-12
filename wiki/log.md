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
