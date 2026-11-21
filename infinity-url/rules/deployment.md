---
paths:
  - 'compose*.yml'
  - 'compose*.yaml'
  - 'container/**'
  - 'Dockerfile'
  - 'Containerfile'
  - '.github/**'
---

# Deployment

Hosts, Traefik, secrets and backups are in `docs/08-setup.md`.

## One product, two entry points

| Host | Serves |
| --- | --- |
| `iurl.fr` | Redirects only. No marketing page, no third-party script, no cookie |
| `url.infinityproject.fr` | Dashboard and marketing |
| `api.url.infinityproject.fr` | Public API |

`iurl.fr` never holds a cookie. The dashboard's refresh cookie is `__Host-`,
host-only on the API host, `HttpOnly`, `Secure`, `SameSite=Lax`. Never a `Domain`
cookie.

## Traefik

- Traefik is shared and already running: the project attaches to its external
  network, it does not ship its own.
- Customer domains are issued **on demand**, gated by `GET /internal/tls/ask`
  requiring `X-Internal-Token`. The endpoint fails closed.
- ACME **staging** everywhere except production. The production rate limit is 50
  certificates per registered domain per week.

## Containers

- One autonomous compose project: own volumes, own networks, own names. Moving it
  to its own machine must be `compose up` plus a DNS switch.
- `postgres` and `valkey` are on no exposed network and publish no port.
- Memory limits per service are mandatory. A traffic spike on the shortener must
  not take down the other projects on the same machine.

## Logs

journald with `MaxRetentionSec=72h`. Docker's `local` or `json-file` driver
rotates by size only, which does not guarantee the 72 hours the privacy policy
promises.

## Secrets

| Secret | Rule |
| --- | --- |
| `APP_KEY` | `.env`, 600. Rotation loses encrypted TOTP secrets |
| `BLIND_INDEX_KEY` | Distinct from `APP_KEY`, **never rotated** |
| Analytics salt | File `/secrets/analytics.salt`, not in the database, **not in backups** |
| `TRAEFIK_INTERNAL_TOKEN` | Shared with Traefik only |

`.env.example` is committed. `.env` never is.

## Images

Built in CI, pulled by tag from GHCR, never built on the server. Rollback is
`TAG=<previous> compose up -d`. Migrations run as a separate one-off command
before `up`, never inside the entrypoint.
