# Index

## Sources
- [[Source---LLM-Wiki-Idea-File]] — Founding document describing the LLM Wiki pattern for persistent, LLM-maintained knowledge bases (2026-04-10)
- [[Source---Entra-ID-OAuth-Reference]] — Comprehensive FAQ-style guide to Microsoft Entra ID, OAuth 2.0, JWT, SPNs, and permissions (2026-04-10)
- [[Source---Entra-ID-Audience-Scopes-Deep-Dive]] — Socratic deep dive: how aud is derived, api:// naming, scopes, two-registration pattern, PKCE, Redirect URIs (2026-04-10)

## Entities
- [[Vannevar-Bush]] — Engineer who proposed the Memex (1945), spiritual predecessor to the LLM Wiki
- [[Obsidian]] — Markdown note-taking app serving as the wiki's browsing layer
- [[qmd]] — Local markdown search engine with BM25/vector hybrid search
- [[Microsoft-Entra-ID]] — Microsoft's cloud identity service (formerly Azure AD), the authorization server for OAuth flows
- [[MSAL]] — Microsoft Authentication Library: JavaScript library that orchestrates all token requests from the React frontend

## Concepts
- [[LLM-Wiki-Pattern]] — Core methodology: LLM incrementally builds a persistent, compounding wiki instead of RAG
- [[RAG]] — Retrieval-Augmented Generation: the standard approach this pattern improves upon
- [[Memex]] — Vannevar Bush's 1945 vision for a personal knowledge device with associative trails
- [[Index-Based-Navigation]] — Using a curated index file for page lookup instead of embeddings
- [[OAuth-2.0-Authorization-Code-Flow]] — Standard grant type for web apps: 10-step traffic path from browser to token
- [[JWT]] — JSON Web Token: signed, self-contained token format with header/payload/signature
- [[Service-Principal]] — Runtime instance of an App Registration, one per tenant
- [[App-Registration]] — Blueprint/definition of an application in Entra ID; covers two-registration pattern for frontend + backend
- [[JWKS]] — Public key endpoint for local JWT signature verification
- [[Scope]] — Named permission unit in OAuth: defines what action a token holder is allowed to perform; checked by Entra ID at issuance and backend on every request
- [[PKCE]] — Proof Key for Code Exchange: hash-based mechanism public clients use instead of client_secret
- [[Public-vs-Confidential-Client]] — Fundamental OAuth split: whether an app can safely store a secret (drives two-registration pattern)
- [[Redirect-URI]] — Pre-registered URL whitelist in App Registration; Entra ID only sends auth codes to registered addresses

## Analyses
