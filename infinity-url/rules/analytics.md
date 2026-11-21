---
paths:
  - 'app/Actions/**'
  - 'app/Jobs/**'
  - 'app/Services/**'
  - 'app/Support/**'
  - 'routes/**'
---

# Analytics pipeline

The full sequence is in `docs/11-pipeline.md`. This rule concentrates the parts
where a mistake is silent: a wrong counter does not crash, it lies.

## Split between request and job

Everything that needs the IP happens **in the request**, because the IP is never
persisted:

| Computed in the request | Passed to the job |
| --- | --- |
| `hash_d`, `hash_w` (HMAC with the salt) | Both hashes |
| Country (v0.2) | `country` or `null` |
| Device class, bot flag | `device`, and the job is skipped for bots |
| Date, UTC | `date` as `Y-m-d` |

The job payload contains **no personal data**: no IP, no user-agent, no
destination. A failed job can be logged without leaking anything.

## Idempotence

A retried job must not increment clicks twice. The dedup key protects uniques
only; clicks need their own guard:

```php
if (! Cache::add("job:{$this->job->uuid()}", 1, now()->addHours(6))) {
    return;
}
```

`$this->job->uuid()` is stable across retries. Never remove this guard.

## Counting

```php
$uniqueDay  = Cache::add("u:d:{$hashD}", 1, $endOfUtcDay);
$uniqueWeek = Cache::add("u:w:{$hashW}", 1, $nextSaltRotation);
```

Then one atomic upsert per table. Never `select` followed by `update`: two
concurrent jobs would lose an increment.

## Bots and prefetches

Not counted: identified bots, `HEAD`, `Sec-Purpose: prefetch`,
`Purpose: prefetch`, `X-Purpose: preview`. Counted with `device = unknown`: a
request without a user-agent. Only exclude what is *identified* as a bot, never
what is unknown.

## Failure modes

> Fail **open** on the redirect, fail **closed** on the counters.

| Failure | Redirect | Analytics |
| --- | --- | --- |
| Valkey down | Served from the database | Jobs retried |
| Database down | Served from cache | Jobs retried |
| Queue unavailable | Continues | Event lost, metric, alert |
| Salt missing | Continues | Clicks counted, uniques skipped, `CRITICAL` |
| Redis cache miss storm | `SETEX` under singleflight lock | — |

Never make a visitor wait because a statistic cannot be written.

## Date

Captured in the request and carried in the payload. The job never calls `now()`
for the bucket date: a job running at 00:00:03 for a click at 23:59:58 must land
on the previous day.
