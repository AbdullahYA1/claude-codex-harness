# Harness memory — how the loop itself behaves

Loop-level learnings only: Codex behaviour, prompts that worked or failed, Validator quirks, tooling traps. Where other facts go:
- **Feature or domain facts** → the matching `knowledge/*.md` file.
- **Cross-session user/project facts** → Claude Code's auto-memory, per the system memory rules.

One bullet per learning: `- YYYY-MM-DD — <fact>. **Why it matters:** … **Apply:** …`. Update a bullet in place instead of adding a near-duplicate. Delete bullets that turn out wrong.

The seed bullets below ship with the harness. Replace or delete them as your own setup proves otherwise.

## Codex CLI

- seed — The model catalog is served per client version: an old `codex` may not list the model named in `config.toml`, and falls back silently. **Apply:** keep the preflight version check.
- seed — A permission rule `Bash(codex exec:*)` matches the literal command prefix: `cd X && codex exec`, `env … codex exec` and `VAR=… codex exec` don't match. **Apply:** always start the command with `codex exec --cd <repo>`.
- seed — `--sandbox workspace-write` blocks network and writes outside `--cd`. On some older Linux kernels / WSL builds it can hang at start. **Apply:** use it by default; if it hangs, check `codex --version` and the kernel before reaching for the bypass flag.
- seed — If an IDE extension also runs Codex against the same `~/.codex`, it can hold a lock on the state DB and `codex exec` deadlocks. **Apply:** give the harness its own `CODEX_HOME` (see README) and refresh its `auth.json` copy in preflight.
- seed — `codex exec … | tail -60` shows nothing until exit. **Apply:** `tee` the output to a log and run it in the background, so the user can `tail -F` it.
- seed — Transport errors for optional MCP servers and "failed to refresh available models" warnings in Codex output are usually harmless. **Apply:** ignore them unless the run has no final answer.

## Prompts to Codex

- seed — Codex can read files outside `--cd` in workspace-write mode. **Apply:** pass knowledge file **paths** ("read …/knowledge/<file>.md first") instead of pasting their contents into the prompt.
- seed — Codex runs inside the sandbox, not inside your containers. **Apply:** tell it which checks it can run itself (syntax check, linter, `git diff --check`) and leave container or network tests to the Validator and orchestrator.
- seed — Given only the path to `code-quality.md` plus the "name what you skipped" line, Codex tends to self-report which slop items it avoided. **Apply:** keep that line; it makes the diff review faster.

## Validator

_(no learnings yet)_
