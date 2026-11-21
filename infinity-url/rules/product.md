---
paths:
  - '*'
---

# Infinity URL

Privacy-first URL shortener. The link domain is `iurl.fr`. The product belongs to
Infinity Project.

## Repositories

| Repository | Contains | Stack |
| --- | --- | --- |
| `infinity-url-api` | Redirect route, REST API, analytics pipeline, jobs | Laravel 13, PHP 8.4, PostgreSQL 17, Valkey 8 |
| `infinity-url-web` | Dashboard, marketing pages, QR rendering | Nuxt 4, Vue 3 |

The dashboard is a client of the **public** API, exactly like `iQuiz` or a third
party. There is no private endpoint reserved for the dashboard. If a screen needs
a datum, the datum goes through the documented API.

## Read the dossier before inventing anything

`docs/` holds the design. It is not background reading, it is the specification:

| File | Content |
| --- | --- |
| `01-decisions.md` | Every arbitrated decision, numbered `D-xx` |
| `07-schema.md` | The database model, as SQL |
| `10-api.md` | The HTTP contract |
| `11-pipeline.md` | From a click to a counter |
| `12-uml.md` | State machines and sequences |
| `09-tests.md` | How this project is tested |
| `08-setup.md` | Hosts, Traefik, compose, secrets, packages |

If a task contradicts a decision, **stop and name the decision**. Do not
implement a third option silently, and do not re-litigate a settled point.

## What does not exist, and must not be added

- Anonymous link creation.
- A link-in-bio page.
- Password-protected links (deferred).
- Referer, user-agent or IP stored anywhere.
- A click-level event table.
- Billing, teams, roles beyond `owner`, SSO.
- A third-party script on `iurl.fr`.
- A session list exposed to the user.
- A browser extension that observes navigation. A `302` with no body cannot host a
  content script: detecting our redirects would require `webRequest` on every
  site. Anonymous reporting already exists without an extension.

Each of these is a product decision, not an implementation detail. Stop and ask.
