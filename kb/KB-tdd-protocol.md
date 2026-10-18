---
name: KB-tdd-protocol
id: KB-tdd-protocol
date: 2026-06-09
status: active
parent: ../README-kb.md
---

# KB: TDD Protocol

## Red/Green Cycle Rules

Each cycle maps to exactly ONE scenario (GIVEN/WHEN/THEN) from the OpenSpec change's spec deltas.
A scenario is the acceptance criterion (AC). The test name must contain the scenario name.

```
For each scenario in openspec/changes/<change-id>/specs/:
  1. RED   — Write the failing test first. Run test suite. Confirm it fails.
  2. GREEN — Write minimum code to make the test pass. No extra logic.
  3. REFACTOR — Clean up. Run tests again. Confirm still green.
  4. COMMIT — See batch rules below.
```

Never write implementation code before the failing test exists.
Never commit a red test.

## Commit Batch Rules

**Rule: 1 commit = 1 scenario (AC)**

Format:
```
<type>(<change-id>): <what this scenario does>

Scenario: <exact scenario name from the spec delta>
Tests: <test file(s) added or modified>
```

Example:
```
feat(add-login-endpoint): add user authentication endpoint

Scenario: User can POST /api/auth/login with email+password and receive JWT
Tests: src/test/kotlin/AuthControllerTest.kt
```

Allowed types: `feat`, `fix`, `refactor`, `test`, `db`, `chore`

## Batch Size Guidelines

| Scenario | Commit strategy |
|---|---|
| 1 AC = 1 small function | 1 commit |
| 1 AC = db migration + backend | 2 commits (db first, backend second) |
| 1 AC = backend + frontend | 2 commits (backend first, frontend second) |
| Schema change touches 10+ files | Still 1 logical commit — all in same AC batch |

## jOOQ-Specific TDD Rules

When an AC requires a schema change:

```
1. Write the test AGAINST the new jOOQ classes (will fail to compile — that's the RED)
2. Create migration SQL
3. Run flywayMigrate
4. Run generateJooq
5. Now tests compile — run them (still RED, no implementation yet)
6. Implement backend logic using new jOOQ classes (GREEN)
7. Commit db migration as: db(<change-id>): <migration description>
8. Commit backend as: feat(<change-id>): <feature description>
```

## Stuck Detection

If the same test fails for 3 consecutive fix attempts:
1. Write the failure context (all attempts, error output) in a `[AGENT] stuck` comment on the issue (connector `comment`)
2. Add label `needs-human`, remove `agent-running` (connector `set_labels`)
3. Do not open a PR
4. STOP — do not keep trying

Max retries: 3 per test failure
Max total implementation attempts per AC: 5

## Content projects (type: content)

There are no unit tests. RED = the structure/build check for the scenario fails (e.g. the new page is missing from `src/SUMMARY.md`, or `mdbook build` fails).
GREEN = the check passes. The commit rule (1 scenario = 1 commit) is the same.

## What "Minimum Code" Means

GREEN means the test passes. It does NOT mean:
- The most elegant solution
- Handling every edge case
- Refactoring existing unrelated code

Refactor in step 3 only. Do not scope-creep during GREEN phase.

## Changelog
- 2026-10-13 — v1.1 — AC = OpenSpec scenario; stuck log moves to an issue comment; content projects section
- 2026-06-09 — v1.0 — initial version
