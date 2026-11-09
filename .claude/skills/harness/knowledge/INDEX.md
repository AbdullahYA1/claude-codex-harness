# Project knowledge index

<!-- TODO: replace this paragraph with your project prompt. Remove every TODO marker once filled. -->
You are a senior engineer on **TODO: project / domain name**: TODO one sentence on what the system does, across **TODO: repo-a** (`TODO: /path/to/repo-a`) and **TODO: repo-b** (`TODO: /path/to/repo-b`). The user does not need to re-explain the system. This index tells you which knowledge file to open.

## How to use this index

1. Read the task. Match its words against the **Load when** column below.
2. Open **only** the matching files (usually 1–3). Always add `00-system-map.md` when the task spans more than one component or repo. Never load every file "to be safe".
3. Each file has the same sections: *Load when · Verified · Facts · Invariants · Gotchas · Verify · Sources*. **Invariants** are the rules a diff must keep. Pass them to Codex and to the Validator.
4. Files are dated summaries, not the code. Before changing anything, read the code the file points to. If code and file disagree, the code wins: fix the file in the Learn step.
5. Go to **Sources** (repo docs, fix logs, runbooks) only when the file says the detail lives there.

## Routing table

| File | Load when the task mentions… |
|---|---|
| [code-quality.md](code-quality.md) | **always, for any code change**: the clean-code / no-slop / efficiency standard Codex writes to and the Validator reviews against |
| [00-system-map.md](00-system-map.md) | which component owns X, cross-repo flow, routes, environments |
| [local-env-and-testing.md](local-env-and-testing.md) | running or calling things locally, "change had no effect", tests hang/fail, test commands, auth for local requests |
| TODO `<feature>.md` (copy `_TEMPLATE.md`) | TODO: the words, class names, endpoints, error messages that should route here |

Common pairings: TODO (e.g. "a new API filter → `search-filters` + `search-indexing`").

## Rules that apply to every task

- **Git**: never commit, stage, push, stash, checkout or reset without explicit approval. Leave changes unstaged. Codex must not run git writes either.
- **Production over local**: never size, validate or tune performance on unrepresentative local data.
- **Scope**: TODO (out-of-scope repos or areas; areas that take bug fixes only).
- **Quality gates**: TODO (e.g. `npm run lint` → `npm run typecheck` → `npm test`; note pre-existing failures and compare before vs after).
- **Fix logs**: TODO (e.g. after a fix, append to the repo's `.claude/FIX_LOG.md`, newest first) — or delete this line.
- **Flags**: TODO (e.g. rollout behaviour goes behind feature flags; tests cover flag on **and** off) — or delete this line.

## Deeper sources (open only when pointed to)

- TODO: repo docs, ADRs, runbooks, repo-specific skills, fix logs.

## Keeping this current

When a task proves a file wrong or adds a reusable fact, edit that file in the harness Learn step: change the fact in place, bump **Verified** (date + branch/commit), and keep it short. Add a new feature file only for a new feature area, then add its row here. Stale knowledge is worse than none.
