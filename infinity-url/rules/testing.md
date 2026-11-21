---
paths:
  - 'tests/**'
  - '**/*.spec.ts'
  - '**/*.test.ts'
---

# Testing

The full strategy is in `docs/09-tests.md`. Three rules summarise it.

## 1. Test the promises, not the getters

These tests are the product. They must exist from the first commit and never be
weakened:

| Invariant | Test |
| --- | --- |
| No cookie on the redirect | `GET /{slug}` has no `Set-Cookie` |
| No IP column anywhere | Schema introspection |
| No click-event table | Schema introspection |
| Email never readable in the database | `users.email` holds ciphertext |
| Blind index never serialized | `email_hash` absent from every response |
| `APP_KEY` rotation keeps the index | Changing `app.key` does not change the hash |
| Dedup key is link-scoped | Same IP, two `link_id` → two different keys |
| A job retry does not double count | Replay the job, clicks unchanged |
| Salt is destroyed on rotation | Old salt no longer readable |
| No route without an ability | Route inspection |
| Export is allowlisted | `GET /me/export` keys match the inventory |

If one of these fails, the code or the document is wrong — **never the test**.
Changing a test of this table requires changing `docs/09-tests.md` in the same
change.

## 2. Real services

PostgreSQL and Valkey, from `compose.test.yml`. Never SQLite, never a mocked
cache: the TTL behaviour, the singleflight and the unique constraints *are* what
is being tested.

## 3. Small and honest

- 100% coverage required on four modules only: dedup key, slug generation,
  retention, redirect route.
- No global coverage target.
- Three E2E journeys, not thirty: signup to first click, custom domain with TLS
  (ACME **staging**), API call with a bearer token.
- No mutation testing in v1.

## Naming

Pest. One file per behaviour, named after the behaviour, not after the class.
