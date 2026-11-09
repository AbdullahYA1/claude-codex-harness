# Background jobs & queues (example-api)

<!-- EXAMPLE of a filled-in knowledge file, for a fictional Laravel app called example-api.
     Copy the shape, not the facts. -->

**Load when:** queue jobs, workers, `queue:retry`, `failed_jobs`, `retry_after`, job timeouts, nightly rebuild jobs, duplicate job runs, write spikes on the database at night.
**Verified:** 2026-01-15 (main `a1b2c3d`).

## Facts

- Queue connection is Redis; most jobs run on `default`. Worker flags (`--tries`, `--timeout`) live in the deploy config, not the repo.
- `config/queue.php` redis `retry_after` = **90 s**. Any job running longer than 90 s is **put back on the queue as a copy while still running**, and another worker runs it in parallel. The framework requires `retry_after` > the longest job timeout.
- The nightly `reports:rebuild` command dispatches one `RebuildReportJob` per account. The job had `$timeout = 3600`, so every long run spawned copies every 90 s.
- `queue:retry` with no id re-queues every `failed_jobs` row at once and deletes the rows. Nothing in code calls it; it is always manual.
- Never page large tables with offset `chunk()`: it is `LIMIT/OFFSET` and slows down quadratically. Use `chunkById()` / `lazyById()`.

## Invariants

1. A job that can run longer than `retry_after` must be safe to start twice: `WithoutOverlapping($key)->dontRelease()->expireAfter($timeout)` (the default re-queues the duplicate and keeps the loop going).
2. Long jobs set `failOnTimeout = true`; a timed-out run fails once and never re-runs.
3. Unlimited `tries` only together with `maxExceptions` and no `release()` path.
4. Nightly-rebuild jobs carry a dispatch timestamp and skip when older than their max age, so a `queue:retry` replay does nothing. A missing timestamp (old payload) still runs.

## Gotchas

- Killing workers doesn't remove jobs: reserved entries come back after `retry_after`. Delete the payloads from `queues:default`, `:reserved`, `:delayed` and `queue:forget` the failed rows.
- Job settings are serialized into the payload at dispatch time: after changing them, purge the old payloads or old copies keep the old behaviour.

## Verify

`php artisan tinker` → count `RebuildReportJob` payloads in `queues:default` and `queues:default:reserved` during a run (should be ≤ 1 per account). Tests: `vendor/bin/pest tests/Unit/Jobs/RebuildReportJobTest.php`.

## Sources

`docs/runbooks/queues.md`; `.claude/FIX_LOG.md` entry 2026-01-15.
