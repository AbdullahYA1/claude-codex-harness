# Harness task tracker

The orchestrator owns this file. One entry per `/harness` task. Update it at every loop step so an interrupted session can resume from here.

Status values: `planned` → `delegated` → `validating` → `passed` | `failed` | `blocked` → `done`

## Active

_(none)_

<!-- Entry template — copy into Active:
### T-NNN — <short title>  ·  status: planned  ·  opened YYYY-MM-DD
- **Repo(s):** <path(s)>  ·  branch: <branch>  ·  baseline: `git status` clean? / pre-existing dirty files: <list>
- **Goal:** <one sentence>
- **Acceptance criteria:**
  1. ...
- **Knowledge loaded:** knowledge/<file>.md, ...  (invariants passed to Codex + Validator)
- **Work items:**
  - [ ] W1 (<repo>): <item>
- **Cycles:** C1 delegated → Validator FAIL (findings 1,3) · C2 → PASS
- **Files changed:** <paths, from git diff --stat>
- **Learned:** <knowledge file updated / MEMORY.md line / none>
-->

## Done (newest first, keep the last 20)

_(none)_
