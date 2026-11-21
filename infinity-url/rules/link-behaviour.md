---
paths:
  - 'app/Actions/**'
  - 'app/Http/**'
  - 'app/Models/**'
  - 'app/Services/**'
  - 'routes/**'
---

# Link behaviour

Ten arbitrated behaviours. Each one has a status code and a test; none is
negotiable at implementation time.

## 1. Renaming a slug

The old slug is inserted into `slug_tombstones` and answers `410` from then on.
It is never released: it may be printed on a QR code or shared already.

## 2. Deleting a domain that still has links

`409 Conflict`, code `domain_has_links`. No cascade. Deleting a domain must
never silently destroy links and their statistics.

## 3. Deleting an account

Soft delete immediately, all tokens revoked. The email stays occupied during the
grace period, so nobody else can take the address. Logging in during the 30 days
**reactivates** the account. After that, physical deletion: workspace, links and
statistics cascade, each slug copied into `slug_tombstones`, the email becomes
available again.

## 4. Country

Not in v1. The v0.1 dashboard shows clicks and device class only. The `country`
dimension exists in the schema and stays empty until v0.2.

## 5. `/internal/tls/ask`

Requires `X-Internal-Token`, compared with `hash_equals`. Answers `200` only for
a domain that exists, is verified, is active and is not deleted. Anything else is
`404`. Never expose this route through Traefik.

## 6. Link quota

Enforced in a service called by the creation action, inside a transaction with
`lockForUpdate` on the workspace row. Not in the controller: the dashboard and
the API must share the same path, and a double click must not produce a sixth
link.

## 7. Reactivating a link

A `disabled` link (expired) is reactivated by pushing `expires_at` forward. A
`blocked` link (abuse) is **never** reactivated by the user: only a manual
operator action. A `deleted` link cannot be reactivated at all.

## 8. `?q=` on the link list

Matches `slug` and `destination` with `ILIKE`, minimum two characters, scoped to
the current workspace. Never the email, never a hash column.

## 9. Transactional emails

Sender `Infinity Project <noreply@infinityproject.fr>`, `Reply-To`
`contact@infinityproject.fr`. Four messages only: verification, password reset,
new-login alert, link blocked. Minimal HTML plus a plain-text part. No image, no
tracking pixel.

## 10. Destination check at creation

Refuse on proof of danger, never on absence of proof of safety:

| Result | Behaviour |
| --- | --- |
| Domain on a blocklist | `422`, `destination_blocked` |
| Private, loopback or metadata address | `422`, `destination_private_address` |
| Non-HTTP scheme, credentials in URL | `422` |
| Unreachable (timeout, DNS) | **Accepted**, logged, counted in a metric |
| Reachable and clean | Accepted |

Typosquatting is **deterministic, never a model**: a curated set of ~200 protected
brands, a job that precomputes every variant (substitution, deletion, insertion,
transposition, homoglyph, TLD swap, combosquat, concatenation) into an indexed
table, and one lookup at creation. Add two at-runtime checks: the brand appearing
in a **subdomain** of a third-party registrable domain, and **mixed Unicode
scripts**. Always carry a citable reason — "one letter away from `laposte.fr`".

**A typosquat alone never blocks.** It sends the link to manual review, because
the domain may be a legitimate business. It blocks only combined with another
signal: domain younger than 30 days, blocklist hit, or recent reports. Homoglyph
and mixed-script hits block on their own.

**The brand list is a booster, never the foundation.** The generalisable signal is
"young domain + a login path", not "looks like a brand". There is **no popularity
signal**: a rank measures inbound links, not trustworthiness, it duplicates domain
age, and it mis-ranks low-visibility legitimate brands (a retail bank can sit at
rank 963 189). The Majestic Million was used once, by hand, to build the seed
list; it is never fetched by a job, versioned, or redistributed. The list stays short (~120 entries), sourced and dated, because
forgetting a brand is harmless while adding a wrong one blocks a legitimate link.
New candidates are proposed by a weekly job over the phishing dumps and accepted
by a human — never applied automatically.
