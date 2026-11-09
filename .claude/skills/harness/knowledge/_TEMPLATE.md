# <Feature area> (<repo>)

<!-- Copy this file to knowledge/<feature-area>.md, fill it in, and add a routing row in INDEX.md.
     Keep it under ~60 lines: a dated map that tells the reader where to look, not a copy of the code. -->

**Load when:** <the words a task would use: feature names, class/function names, endpoints, error messages, symptoms>.
**Verified:** YYYY-MM-DD (<branch> `<short-sha>`).

## Facts

- <How it works, in the fewest words that still prevent a wrong change. Point to `path/to/file.ext:line`.>
- <Config values and where they really live (repo file vs deploy config vs environment).>
- <Data flow: who writes this, who reads it, what rebuilds or overwrites it.>

## Invariants

<!-- Numbered, so the Validator can report "held / broken / not touched" per item. Each one is a rule a diff must keep. -->
1. <A rule a diff must keep, e.g. "Every new filter is added in both the query builder and the index definition.">
2. <...>

## Gotchas

- <Traps that cost someone hours: silent no-ops, caches that hide changes, misleading names.>

## Verify

<How to prove a change works: the test command, a request to send, a query or metric to look at.>

## Sources

<Repo docs, fix log entries, runbooks, incident write-ups where the detail lives.>
