# Local environment & testing

<!-- TODO: fill this in. The Validator reads it for test commands and safety preconditions, so keep the commands copy-pasteable. -->

**Load when:** running or calling anything locally, a change "has no effect", tests hang or fail strangely, auth for local requests, seeding fixtures.
**Verified:** YYYY-MM-DD.

## Containers & ports

| Thing | Where |
|---|---|
| <app> | <container name>, `http://localhost:<port>` |
| <database> | <container>, host port <n>, connection string notes |
| <cache / queue> | <container>, db/prefix |

## "My change did nothing"

1. <Long-running servers that keep code in memory: how to really reload (e.g. restart the container).>
2. <Env precedence: container env vs `.env` files; what needs a recreate.>
3. <Caches to flush, and the exact command.>

## Auth for local requests

<Where test tokens/users come from, how to force a feature flag locally.>

## Tests

| Repo | Command (run from) | Notes |
|---|---|---|
| <repo-a> | `<lint>` → `<typecheck>` → `<test>` (inside `<container>` / on host) | <known pre-existing failures: compare before vs after> |
| <repo-b> | `<...>` | <...> |

## Safety preconditions (the Validator checks these before running anything)

<!-- List any check that must pass before a test run, and what to do if it fails. Example:
- <repo-b>: `grep TEST_DB_URL phpunit.xml` must match. If it doesn't, do NOT run tests (they would hit the real dev database); report a BLOCKER. -->
- <none yet>

## Data reality

<How big local data is vs production, and where to get real numbers instead (read-only staging dashboards, metrics, logs). Never size or tune on local data.>

## Sources

<Setup docs, docker-compose files, runbooks.>
