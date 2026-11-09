# Code quality standard — clean, readable, efficient, no AI slop

**Load when:** **every task that changes code.** It isn't routed: the loop always passes it to Codex and the Validator.
**Verified:** ship version. Distilled from widely used public agent skills (see Sources). Language-agnostic; examples are illustrative.

**Precedence:** the user's instructions and standing rules, then repo rules, then this file, then any other skill (full order: `SKILL.md` → "Skill interop"). **The repo overrides.** Documented repo rules (`AGENTS.md`, `CLAUDE.md`, contributing guides, review checklists) beat this file. Rules a formatter or linter enforces are left to the tool, not re-reviewed by hand.

## 1. Before writing: the ladder

Stop at the first rung that holds:
1. **Does it need to exist?** Build only what the goal asks for. Speculative needs are skipped and named in one line of the report.
2. **Does it already exist here?** Reuse the helper, repository, DTO, enum, util or module a few files over. Re-implementing what exists is the most common slop.
3. **Framework or stdlib?** Prefer what the language, framework and database already provide over custom code or a new dependency.
4. **The minimum code that works**, in the place every caller routes through. A bug fix goes in the shared function, not in each caller (grep the callers first).

Understand first (Chesterton's fence): know the code's callers, edge cases, tests and history before changing or deleting it. A small diff in the wrong place is a second bug.

## 2. Readability

- Names say what a thing **is** in domain words (`invoiceTotal`, `nightsCount`, `blockedUserIds`), not how it is computed or generic words (`data`, `result`, `handle2`, `tmp`). Booleans read as questions: `isActive`, `hasLicense`.
- One function = one level of abstraction and one reason to change. Guard clauses and early returns instead of nesting deeper than two levels.
- No nested ternaries, no clever chains a reader must execute in their head. Name the intermediate step.
- Avoid boolean flag parameters that switch behaviour in new code; prefer two named functions or an enum. Where the codebase already uses them as configuration, follow that pattern and pass them as **named/keyword arguments** where the language allows.
- Comments explain **why** (a constraint, a trap, a decision, a ticket): `// the nightly job rebuilds this record, so re-attach the field here`. Never comment what the code already says.
- Match the surrounding code: its naming, comment density, error style and layering. Consistency beats personal preference.

## 3. AI slop — never ship these

Each item below is a **BLOCKER** in validation unless the goal explicitly asks for it:
1. Features, refactors, renames or "improvements" beyond the goal; reformatting untouched lines.
2. Comments, docstrings or type annotations added to code that wasn't otherwise changed; comments restating the code; `// removed …`, `// old version` or `// TODO: maybe` breadcrumbs.
3. Defensive `try/catch` on trusted internal paths, or a catch that swallows the error and returns a default (`catch (e) { return []; }`). Handle errors where something can actually fail (I/O, HTTP, database, queue) and either rethrow or log with context.
4. Validation or null checks for states the types and callers already rule out, duplicated in every caller instead of once at the boundary.
5. A one-implementation interface, a factory for one product, a config key for a value that never changes, a wrapper that only forwards calls (the deletion test: if deleting it loses nothing, it shouldn't exist).
6. Backwards-compatibility shims nobody asked for: re-exports, renamed `_unused` vars, dead branches "just in case", feature switches with one side unreachable.
7. Tests edited or deleted to make the change pass; assertions loosened; tests that assert the mock instead of the behaviour.
8. Debug leftovers: `print`/`console.log`/`var_dump`/`dd()`, debug-level log noise, commented-out code.
9. Hard-coded secrets or tokens, reading environment variables deep in application code instead of through the config layer, magic numbers that need a name.
10. Duplicating a predicate or formula that already exists: reuse it, so the rule can't drift.

## 4. Efficiency (correct first, then cheap)

- No queries inside loops (N+1): eager-load, batch by ids in chunks, or join. One query per item is fine only when the set is small and bounded and the reason is documented.
- Stream large reads (cursors, keyset pagination, chunked jobs). Never hold millions of rows in memory. Push aggregation into the database when it can do it.
- Database pipelines and queries: filter as early as possible, select/project only the fields used, avoid sorts the index can't serve, index every field you filter on. Prefer keyset (`WHERE id > ?`) over offset paging on large tables.
- Cache only with a key that includes everything the result depends on (user, tenant, flag variant, locale, client type), and a TTL that matches how stale it may be. A cache key missing an input is a data leak.
- Long-running servers and workers: no request state in static/global variables or singletons; memoise per request.
- Don't optimise blind: measure on production-like numbers, never on a tiny local dataset (see `local-env-and-testing.md`).

## 5. Design (when the change adds structure)

- Prefer **deep modules**: a small interface hiding real behaviour. Avoid shallow pass-through classes whose interface is as big as their body.
- **One adapter means a hypothetical seam; two adapters make a real one.** Don't add an interface until the second implementation exists (tests may use fakes).
- Accept dependencies (injection) instead of constructing them inside; return results instead of mutating hidden state. This is what makes code testable through its interface.
- Some duplication is better than the wrong abstraction. Extract on the third real use, not the first.

## 6. Diff discipline

- The diff contains only what the goal needs. A refactor is a separate work item from a feature or fix.
- Behaviour-preserving changes keep the existing tests passing **unmodified**.
- Each change is small enough to review in one sitting. Delete dead code you made dead; don't delete code you don't understand.

## 7. Tests

- One focused test per behaviour, through the public interface. Name the test after the behaviour it asserts.
- Flag-dependent code: test the flag **on and off**. Time-dependent code: freeze the clock.
- New behaviour or a bug fix → test first (red → green): the test fails before the change and passes after. For a bug, that test is the regression test.
- No new behaviour (rename, config, docs, a refactor already covered by tests) → no new test; YAGNI applies to tests too.
- Tests use the repo's test framework, never ad-hoc assert scripts or throwaway demo files.

## 8. Validator smell baseline (judgement calls → WARN, never BLOCKER alone)

Long function · long parameter list · duplicated logic · feature envy (a method mostly using another class's data) · shotgun surgery (one change touching many unrelated places) · speculative generality · primitive obsession for domain values (money, dates, durations as loose ints/strings where a type exists) · mysterious names · deep nesting · large class. Cite the hunk and suggest the fix.

## Sources

Ideas adapted from public agent-skill collections: mattpocock/skills `code-review` (two axes: standards vs spec; "the repo overrides"; smells are judgement calls) and `codebase-design` (deep modules, deletion test, seams); addyosmani/agent-skills `code-simplification` (behaviour-preserving, clarity over cleverness, scope to what changed); rohitg00/pro-workflow `deslop` (slop pattern list); oh-my-claudecode `ai-slop-cleaner` (deletion-first, tests first). John Ousterhout's *A Philosophy of Software Design* for deep modules.
