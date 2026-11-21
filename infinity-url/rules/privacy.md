---
paths:
  - 'app/Actions/**'
  - 'app/Http/**'
  - 'app/Jobs/**'
  - 'app/Models/**'
  - 'app/Services/**'
  - 'app/Support/**'
  - 'routes/**'
  - 'database/**'
---

# Privacy invariants

These are not guidelines. They are the product. A change that breaks one of them
turns the published privacy policy and the legitimate-interest assessment into
false statements. If a task seems to require breaking one, **stop and say so**.

## Never stored

| Forbidden | Where it would appear |
| --- | --- |
| IP address | Any column, any application log, any job payload |
| User-agent | Any column, any log |
| Referer | Any column |
| A row per click | Any table |
| A visitor identifier usable across links | Any cache key or column |

The IP exists only in memory, during the request. It is read, reduced, and
forgotten before the response is sent.

## The two fingerprints

Both are computed inside the request, because the IP is not available afterwards:

```php
$base  = $ip . '|' . $link->id;
$hashD = hash_hmac('sha256', $base . '|' . $date, $salt); // daily uniques
$hashW = hash_hmac('sha256', $base, $salt);               // weekly uniques
```

- The **link id** is in the payload. Never the slug, never the destination.
  Without it, a visitor who clicks two different clients' links produces the same
  key, and cross-client correlation becomes possible.
- The salt is **weekly**, rotates Monday 00:00 UTC, and is **destroyed** on
  rotation.
- The salt lives in a file (`/secrets/analytics.salt`), **never** in the
  database, **never** in a backup. Otherwise "destroyed at rotation" is a lie.

## Nothing on the visitor's device

- No `Set-Cookie` on `iurl.fr`, ever. There is a CI test for it.
- No `localStorage`, no `sessionStorage`, no fingerprinting, no third-party
  script on the link domain.
- Error pages (`404`, `410`, `403`) are static HTML, inline CSS, no JavaScript,
  no external request.

## Logs

- 72 hours maximum, enforced by journald `MaxRetentionSec`.
- Query strings stripped from the access log.
- The application logger never receives `REMOTE_ADDR`.
- Only the Traefik access log sees an IP, and only for security.

## Account data

- `users.email` is encrypted at rest (AES-256-GCM) with a blind index
  `email_hash` (HMAC-SHA256 with `BLIND_INDEX_KEY`, **not** `APP_KEY`).
- The hash column is never serialized, never returned, never accepted as input.
- `totp_secret` is encrypted. Recovery codes are hashed (argon2id).
- `links.destination` and `domains.hostname` stay in clear text: they are public
  by nature. Do not encrypt them "for consistency".
