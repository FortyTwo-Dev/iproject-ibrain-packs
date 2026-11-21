---
paths:
  - 'database/**'
  - 'app/Models/**'
---

# Data model

The schema is in `docs/07-schema.md`, as SQL. It is the source of truth; a class
diagram would only drift from it.

## Rules that must survive every migration

| Rule | Why |
| --- | --- |
| No `ip_address`, `user_agent`, `referer` column | Makes the privacy policy false |
| No `clicks` / `visits` / `events` table | Makes erasure impossible and builds a profile |
| `workspace_id` not null on every business table | Tenant isolation |
| `public_id` (ULID) exposed, `bigint` internal | No enumeration through the API |
| `unique (domain_id, slug)` **including soft-deleted rows** | A slug is never reassigned |
| `slug_tombstones` on hard delete **and on rename** | Protects printed QRs and shared links |

## Links

`deleted_at` is a soft delete: the row stays, so the unique index keeps the slug
occupied and the redirect answers `410`. A hard delete (GDPR erasure) removes the
row and inserts `(domain_id, slug, erased_at)` into `slug_tombstones`.

## Statistics

Three tables, because uniques are **not additive**:

| Table | Grain | Holds |
| --- | --- | --- |
| `link_stats_daily` | `(link_id, date)` | `clicks` and `uniques` |
| `link_stats_weekly` | `(link_id, iso_year, iso_week)` | `uniques` |
| `link_stats_dim` | `(link_id, date, dimension, value)` | `clicks` only |

Never store uniques by country or by device: summing them would exceed the real
total. Never sum daily uniques to get a longer range.

## Accounts

`users.email` is `text` (ciphertext) plus `email_hash char(64) unique`. The cast
`EncryptedWithHash` writes both in one operation and implements `compare()` so a
re-save with the same plaintext is not marked dirty.

The `*_hash` columns are blind indexes: never in `$fillable`, never serialized,
never accepted from a request.

## Managed tables

`personal_access_tokens` comes from Sanctum. Only two columns are added:
`family_id` and `type`. `migrations`, `jobs`, `job_batches`, `failed_jobs` are
Laravel's: do not customize them.
