---
paths:
  - 'app/Actions/**'
  - 'app/Http/**'
  - 'app/Models/**'
  - 'routes/**'
---

# Abuse reporting

The report channel is anonymous by construction, not by promise. Any change that
makes it less anonymous is a bug, not a feature.

## The channel

```
iurl.fr/report/{slug}  ->  302  ->  url.infinityproject.fr/report?slug={slug}
```

`iurl.fr` serves redirects only, including this one. The form lives on the
marketing host. `report` is a reserved slug, so no link can shadow it, and a
two-segment path cannot collide with a slug.

## Never stored

IP, user-agent, referer, session id, rate-limit fingerprint. The row of a report
never carries any of them.

The rate limiter hashes the IP with the current analytics salt under the `report`
context, with a one-hour TTL. **That fingerprint is never written to the report
row.** One hour of memory is the whole cost of spam protection.

## Always `202`

The endpoint answers `202 Accepted` whether the slug exists or not, and returns a
reference. Any other behaviour turns the form into a tool for discovering which
links exist.

## Nothing is automatic

A report never blocks a link. A human qualifies it within 48 hours; the
destination scan is a signal, never a decision. Without that rule, three clicks
would be enough to take down a competitor's link.

## No proof is ever requested

No click token, no "prove you received this link", no email verification before
submitting. Requiring proof destroys anonymity, which is the only reason someone
reports a phishing link.

## The contact email

Optional, encrypted at rest, no blind index (a report is never looked up by
email). It exists to answer the reporter, never to identify them. When it is
absent, the report is fully anonymous and the confirmation page says so.

## Related surfaces

- Footer of `403`, `404` and `410`: the only place a victim lands on our domain.
- `abuse@iurl.fr`, in the terms, the AUP and the WHOIS, plus a DNS TXT record.
- `/report` on the marketing site, indexable.

## What does not exist

- A "report" button on the redirect: it would require an interstitial.
- A browser extension that observes navigation: a `302` with no body cannot host a
  content script.
- Automatic blocking from a report.

## AI-assisted triage (v0.2 only)

A language model may help rank and summarise reports. It never decides.

- **Input is structured signals, never page content**: registrable domain, domain
  age, blocklist hits, redirect hops, TLS age, punycode, mixed scripts, brand in
  subdomain, recent report count. A phishing page is attacker-written text; feeding
  it to a model with tool access is a prompt-injection path into moderation.
- **Read-only tools only.** No `block`, no `report`, no status change. The
  operator decides.
- **Never send** the reporter's free text, their email, or any IP: the endpoint is
  a third party, however European.
- **Optional and off by default** (`AI_TRIAGE_ENABLED=false`). The product works
  identically without it, and the model is interchangeable behind an
  OpenAI-compatible API.
- The model writes a note with reasons; the citable motivation must still come
  from the signals.
