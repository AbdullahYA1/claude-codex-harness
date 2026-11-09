# System map

<!-- TODO: fill this in (or ask Claude: "/harness draft the system map from the repos"). Delete this comment when done. -->

**Load when:** every cross-repo or cross-component task, or when you don't yet know which component owns the behaviour.
**Verified:** YYYY-MM-DD · <repo-a> `<branch>` (`<sha>`) · <repo-b> `<branch>` (`<sha>`). Code beats this file; re-read the code before acting on it.

## Who owns what

| Component | Path | Stack | Owns |
|---|---|---|---|
| <repo-a> | `<path>` | <language, framework, datastores> | <the behaviour and data it is the source of truth for> |
| <repo-b> | `<path>` | <...> | <...> |
| <out-of-scope service> | — | — | **out of scope**; only relevant as <caller/consumer of X> |

## Data flow (the part that bites)

```
<source of truth> ──(how: API / queue / sync job / CDC)──► <derived store>
        │
        ▼
<reader> ── <what it reads, and what can make it stale>
```

## Environments

| Env | Where | Notes |
|---|---|---|
| local | <docker compose / ports> | <data is a small sample: don't size on it> |
| staging | <URL / dashboard> | <how to query it read-only> |
| production | <dashboards / logs, read-only> | <who may touch what> |

## Invariants

1. <A cross-component rule, e.g. "Fields on the derived document must be written by the sync job too, or the next rebuild drops them.">

## Sources

<Architecture docs, ADRs, READMEs of each repo.>
