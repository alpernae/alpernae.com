---
title: "QUERY Is Here: HTTP's Biggest Update in Years"
date: 2026-07-23
tags: ["web", "api"]
---

For decades, developers have faced an uncomfortable tradeoff: send complex search parameters via GET and risk exposing sensitive data in server logs, or use POST and sacrifice cacheability. As of June 2026, RFC 10008 finally closes that gap with the `QUERY` method.

## The Problem: GET vs POST

The core dilemma has always been how to transmit complex query data without giving something up.

**GET** is ideal for retrieving resources — simple searches, static files, filtered listings. Its biggest advantage is cacheability, which reduces server load and speeds up delivery. But GET has hard limits: URLs can only grow so long, and anything in the query string ends up in server access logs, browser history, and analytics pipelines. Not great for sensitive filters.

**POST** solves the length and logging problems by moving parameters into the request body. But POST is not safe or idempotent by default, which means CDNs, proxies, and browsers won't cache it. You can technically force caching on POST responses, but it requires heavy customization and breaks standard caching semantics — rarely worth the complexity.

So we've been stuck: payload capacity vs. caching performance. Until now.

## The Solution: HTTP QUERY

The `QUERY` method (RFC 10008) was designed specifically to bridge this gap. It allows a request body — so you can send large, structured payloads like JSON — while the protocol explicitly defines it as **safe, idempotent, and cacheable**.

That last part is what makes it significant. Because `QUERY` is defined at the protocol level as safe and idempotent, CDNs, load balancers, and browsers can cache responses and retry failed requests automatically — the same guarantees they apply to GET, without any workarounds.

```http
QUERY /search HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "search_text": "foo",
  "page": "1"
}
```

The parameters stay in the body — out of URLs, out of access logs, out of browser history — while the response remains fully cacheable through standard infrastructure.

In short: the payload capacity and clean logging of POST, with the caching and performance benefits of GET.

## What This Means in Practice

- **No more URL length anxiety** — complex, deeply nested filters belong in the body now
- **Sensitive parameters stay private** — search filters won't leak into log aggregators or analytics
- **Standard caching just works** — no custom cache-control gymnastics needed
- **Automatic retries** — proxies and clients can safely retry failed QUERY requests

## Adoption Timeline

The ecosystem takes time to catch up. Web frameworks, WAFs, API gateways, and reverse proxies all need to add explicit support before `QUERY` becomes the default choice. Some will treat unknown methods as errors. For now, check your stack's RFC 10008 support before shipping.

But the direction is clear. As tooling matures, `QUERY` will become the standard for search and filtering endpoints — the right tool that didn't exist until now. Worth adding to your API design thinking today, even if production rollout comes later.
