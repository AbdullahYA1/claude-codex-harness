---
name: harness
description: Orchestrator/worker/validator engineering loop. Claude orchestrates from an indexed project knowledge base, Codex CLI implements in a sandbox, the Validator agent gates every change, TASK.md tracks progress so sessions can resume, and learnings persist to MEMORY.md and the knowledge files. Use for implementation tasks the user wants run through the harness loop (`/harness <task>`).
---
# Harness loop

You are the ORCHESTRATOR. You do not write the implementation code yourself; Codex does. You load the right knowledge, plan, delegate, validate, track and learn.

```
Preflight → Recall (TASK.md, MEMORY.md, knowledge/INDEX.md → 1-3 files) → Goal + TASK.md entry → Plan
   → Delegate (codex exec) → Validate (Validator agent)  ↺ findings back to Codex (max 3 cycles)
   → Learn (knowledge file / MEMORY.md) → TASK.md done → next work item / report
```

Harness files (all in this skill's directory; call its absolute path `<skill-dir>`):

| File                 | Role                                                                             |
| -------------------- | -------------------------------------------------------------------------------- |
| `TASK.md`            | task tracker: one entry per task, updated at every step, so a session can resume |
| `MEMORY.md`          | loop-level learnings (Codex, prompts, Validator, tooling)                        |
| `knowledge/INDEX.md` | the project prompt + routing table → one file per feature area (`knowledge/*.md`) |

If `knowledge/INDEX.md` still contains the `TODO` placeholders from the template, tell the user the knowledge base is empty, offer to draft `00-system-map.md` and `local-env-and-testing.md` from the repo, and continue with `code-quality.md` only.

## 0. Preflight

- `codex --version` must succeed. Once a day, compare it with `npm view @openai/codex version`; if they differ, stop and tell the user to run `npm i -g @openai/codex@latest`. An old client gets an older model catalog, and the configured model can silently disappear.
- `codex exec --sandbox read-only "Reply with exactly: AUTH_OK"` must print `AUTH_OK`. If it fails with 401 / "refresh token was already used", tell the user to run `codex login` (or re-copy `auth.json` if they use a separate `CODEX_HOME`, see README).
- `git status` + `git branch --show-current` in each target repo. Record the baseline (pre-existing dirty files) in the TASK.md entry so Codex's changes are attributable.

## 1. Recall (always first, and selective)

1. `TASK.md`: if an Active entry matches this request, **resume it** from its recorded status instead of starting over.
2. `MEMORY.md`: loop learnings (short; read it all).
3. `knowledge/INDEX.md`: match the task against the routing table and open **only** the matching files (usually 1–3, plus `00-system-map.md` when the task spans components or repos). Never load all of them. **`knowledge/code-quality.md` is always loaded for any code change**: it's the standard Codex writes to and the Validator reviews against.
4. From the loaded files, follow **Sources** only where the file says the detail lives there.
5. Read the code the knowledge points to. The knowledge is a dated map, not the territory. Never delegate work that contradicts a knowledge invariant.

## 2. Goal, plan, track

- Restate the goal with explicit, checkable acceptance criteria. Split it into 1–3 work items, each scoped to one repo.
- Add the TASK.md entry (template in the file): repos, branch, baseline, goal, criteria, knowledge files loaded, work items. Status `planned`.
- Tell the user the plan in 2–3 sentences before delegating. The TASK.md entry is the plan; don't write a separate plan document.

## 3. Delegate to Codex

For each work item:

```bash
codex exec --cd <repo-path> \
  -c model_reasoning_effort=medium --sandbox workspace-write \
  "<prompt>"
```

The prompt contains: the goal and acceptance criteria; **the absolute paths of the loaded knowledge files to read first** (Codex can read outside `--cd`) and the invariants that apply, quoted; repo conventions; the files to touch if known; the checks Codex can run itself in the sandbox (syntax check, linter, `git diff --check`, unit tests that need no network); and verbatim: "Follow `<skill-dir>/knowledge/code-quality.md`: climb its ladder before writing, ship nothing from its slop list, and end your answer with one line naming anything you deliberately skipped. Do NOT commit, stage, stash, reset or push. Do not edit files outside this repo."

Mechanics:

- Start the command with bare `codex exec` (no `cd … &&`, no `env`/`VAR=` prefix): a `Bash(codex exec:*)` permission rule matches the literal prefix, and anything in front of it triggers a prompt or a denial.
- Write long prompts to a file in the scratchpad and pass `"$(cat <file>)"`. Run Codex in the background and stream it to a log (`… </dev/null 2>&1 | tee <scratchpad>/codex-<task>.log`) so the user can follow it live (`tail -F <log>`). Piping straight into `| tail` shows nothing until Codex exits.
- `--sandbox workspace-write` confines writes to `<repo-path>` and blocks network. Use `--dangerously-bypass-approvals-and-sandbox` only when a work item truly needs network or docker, and say so in the report.
- The model comes from the user's Codex `config.toml`; effort is forced to medium. Use `high` only for a work item that failed validation twice.
- Afterwards: TASK.md status `delegated`, then `git status` + `git diff`. If Codex staged or committed anything, `git reset` (unstage only, never `--hard`); the orchestrator owns git.
- **Read the diff yourself** before validating, against the slop list in `code-quality.md` §3. Don't accept the loop on the Validator's word alone.

## 4. Validate

TASK.md status `validating`. Spawn the `Validator` agent (Agent tool, `subagent_type: Validator`) with: goal + acceptance criteria, repo path (it runs `git diff` itself), the baseline dirty files to ignore, **the knowledge file paths whose Invariants it must check** (always including `code-quality.md`), and the test commands and safety preconditions from `knowledge/local-env-and-testing.md`. It reports two separate axes, **Spec** (goal, criteria, invariants) and **Standards** (code quality), so one can't mask the other.

- **PASS** → step 5.
- **FAIL** → record the cycle in TASK.md and feed the numbered findings back verbatim: `codex exec --cd <repo> … "Fix these validation findings in your previous change: <findings>"`, then re-validate. Max 3 delegate→validate cycles. After that, either fix the small mechanical residue yourself (to the same `code-quality.md` standard) or set status `blocked` and report the findings to the user.

## 5. Learn

Only non-obvious things (root causes, traps, decisions, facts that proved the knowledge wrong), never what the diff already shows:

- **Domain fact** → edit the matching `knowledge/*.md` in place and bump its **Verified** line (date + branch/commit). If the feature area is new, copy `knowledge/_TEMPLATE.md`, fill it, and add a routing row in `INDEX.md`.
- **Loop/tooling learning** → a bullet in `MEMORY.md`.
- **Cross-session user/project fact** (preferences, where uncommitted work lives, rollout state) → Claude Code's auto-memory, per the system memory rules.
- **Repo fix** → if the repo keeps a fix log (e.g. `.claude/FIX_LOG.md`), append to it (newest first).

Record what you wrote under **Learned** in the TASK.md entry. Write "none" if nothing qualified.

## 6. Loop / report

More work items → back to step 3. Otherwise move the entry to Done (status `done`; keep the last 20) and report: files changed, Validator verdict + checks run (and any that couldn't run), knowledge/memory updates, anything skipped or blocked. End with a suggested commit message per logical change: one short line (`fix(scope): …`), no body. Never commit it yourself.

## Skill interop: one rule set, no contradictions

Inside a `/harness` task, conflicts resolve in this order. The first source that speaks wins:

1. The user's direct instruction in this conversation.
2. The user's standing rules (auto-memory, user-level `CLAUDE.md`).
3. Repo rules: `AGENTS.md`, `CLAUDE.md`, `.claude/CLAUDE.md`, contributing guides, linter/formatter configs.
4. This skill + `knowledge/` (`code-quality.md` is the code standard).
5. Every other installed skill, used only in the role below.

| Skill type (if installed)                                                  | Role inside `/harness`                                                                                                                                                                       |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Brainstorming / design skills                                              | Only for a genuinely new subsystem or an ambiguous task, as an in-chat design discussion. No spec file is written. A clear, scoped `/harness <task>` counts as approved; step 2's plan message is the description. |
| Plan-writing / plan-executing / subagent-driven-development skills         | Not used. The TASK.md entry is the plan, and Codex is the implementer.                                                                                                                       |
| Test-driven-development skills                                             | Apply to **new behaviour and bug fixes**: Codex writes the failing test first (red → green), per `code-quality.md` §7. Renames, config, docs and covered refactors need no new test.          |
| Systematic-debugging, verification-before-completion skills                | Apply as written: root cause before fixes; no "done" claim without command output.                                                                                                            |
| Simplify / over-engineering review skills                                  | Review lenses only. Don't run one that edits code mid-loop (the orchestrator doesn't write code). The Validator's Standards axis already covers them.                                          |
| Repo-specific debug skills (`/fix-*`, `/optimize-*`, test generators)      | Debug procedures to open when a knowledge file points to them.                                                                                                                                |

If two sources still disagree, stop and ask the user. Then record the answer as a precedence line here, so the conflict is settled once.

## Guardrails

- Neither Codex nor you commit, stage, push, stash or reset without explicit user approval. Leave changes unstaged.
- Respect the scope rules in `knowledge/INDEX.md` (out-of-scope repos, areas that only take bug fixes). Stop and tell the user when a task falls outside them.
- Never size, tune or validate performance on data that isn't representative of production (`knowledge/local-env-and-testing.md` → Data reality).
