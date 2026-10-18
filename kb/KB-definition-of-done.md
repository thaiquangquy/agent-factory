---
name: KB-definition-of-done
id: KB-definition-of-done
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: Definition of Done

## Purpose
Objective checklist used by test-verifier (P-04) and pr-composer (P-05)
to decide whether an implementation is ready for a PR. No subjective judgment.

## Universal DoD (all changes)

- [ ] Every scenario in the change's spec deltas has a check named after it, and the check passes
- [ ] Every task in `openspec/changes/<change-id>/tasks.md` is checked
- [ ] `openspec validate <change-id> --strict` passes
- [ ] The change is archived with the **CLI**: `openspec archive <change-id> --yes` (never the archive skill)
- [ ] Archive check: `openspec/changes/<change-id>/` is gone, `openspec/changes/archive/*-<change-id>/` exists, and `openspec/specs/` changed in the diff
- [ ] All blocking gates for the project (`portfolio.yml` → `gates`) pass locally
- [ ] Commit messages follow KB-tdd-protocol (1 commit per scenario)
- [ ] No debug code, `console.log`, `System.out.println`, or new TODO comments in changed files
- [ ] The PR body follows the KB-workflow-conventions template (scenarios, verification table, risk)
- [ ] No edits to `.github/`, `KB-risk-policy.md`, or `portfolio.yml` (if needed: stop, `risk:high`, `needs-human`)

## Project-Specific DoD

### modern-java (type: content)
- [ ] `mdbook build` succeeds
- [ ] Every new `src/**/*.md` is listed in `src/SUMMARY.md`, outside `<!-- -->` comments
- [ ] markdownlint passes with `.markdownlint.yaml`
- [ ] New chapters follow the layout `src/<topic>.md` + `src/<topic>/*.md`
- [ ] No change to the `master` branch or the deploy workflow

### Code project (template: Gradle + Flyway + jOOQ, React UI)

#### db repo touched
- [ ] Migration file follows naming convention `V{version}__{description}.sql`
- [ ] `./gradlew flywayValidate` passes
- [ ] jOOQ codegen ran successfully (no diff in generated classes that wasn't expected)

#### backend repo touched
- [ ] `./gradlew test` passes
- [ ] `./gradlew integrationTest` passes (if integration tests exist)
- [ ] No raw SQL strings — all queries use jOOQ DSL

#### frontend repo touched
- [ ] `npm test` passes
- [ ] `npx playwright test` passes
- [ ] No TypeScript errors (`tsc --noEmit`)

## DoD Fail Actions

| Failure | Action |
|---|---|
| Check fails | P-03 retries, max 3 attempts per check, then stuck detection |
| Missing check for a scenario | P-03 writes the check before marking done |
| `openspec validate` fails | P-03 fixes the code. If the spec itself is wrong: stop, `needs-human` (never edit spec intent) |
| Archive check fails | P-04 re-runs the CLI archive once, then `needs-human` |
| CI fails on the PR | 2 fix attempts (`@claude fix the failing CI checks`), then `needs-human` |
| jOOQ codegen diff unexpected | HALT, `needs-human` |

## Escalation

If DoD cannot be met after max retries:
1. P-04 posts an `[AGENT] verification failed` comment on the issue with the failure list
2. Adds `needs-human`, removes `agent-running`
3. NEVER opens a PR with failing checks

## Changelog
- 2026-10-13 — v2.0 — OpenSpec validate/archive items; modern-java section; code-project section kept as a template; escalation via labels
- 2026-06-09 — v1.0 — initial version
