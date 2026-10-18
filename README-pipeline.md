---
name: Pipeline idea-to-PR
date: 2026-10-13
parent: ./README.md
---

# Pipeline: Idea → OpenSpec → Merged PR

## Goal
Turn an owner's idea (or a clear ticket) into a merged PR through an OpenSpec change, TDD per scenario, and risk-based review.
Connector: `github` (KB-connector-contract).

## Phase Map

| Phase | Agent | Gate |
|---|---|---|
| 0. Ideation | C-00 chief-engineer | ✅ Gate 1 — owner picks an option |
| 1. Intake (ticket entry only) | P-01 ticket-intaker | — clarification → `needs-human` |
| 2. Spec | P-02 spec-planner | ✅ Gate 2 — spec PR merges (auto for `risk:low`) |
| — | spec-sync workflow | Creates the `agent-ready` issue |
| — | cron dispatcher | Picks the top issue: priority, then oldest; 1 per repo; budget; `depends_on` |
| 3. Implementing | P-03 tdd-implementer | soft — all checks green |
| 4. Verification | P-04 test-verifier | soft — DoD + archive check, max 2 retries |
| 5. PR | P-05 pr-composer | ✅ Gate 3 — blocking CI gates |
| 6. Review / merge | P-06 review-resolver | ✅ Gate 4 — owner review for `risk:high` / `auto_merge: never` |

## Gate Checklists

### Gate 1: Option chosen
- [ ] Exactly 2 options with effort, risk + rule, areas
- [ ] One recommendation
- [ ] Owner replied `A`, `B`, `<letter> with <change>`, or `drop`

### Gate 2: Spec PR
- [ ] `openspec validate <change-id> --strict` passes
- [ ] Every scenario is concrete and checkable
- [ ] `agent.yml` present; risk ≥ KB-risk-policy
- [ ] `risk:low` → auto-merge; `risk:high` → owner approval

### Gate 3: CI (blocking, deterministic)
- [ ] Project gates (`portfolio.yml` → `gates`) pass
- [ ] `openspec validate --all --strict` passes
- [ ] Archive check: change folder archived, `openspec/specs/` changed
- [ ] Advisory only: `/opsx:verify`, reviewer agent, LanguageTool

### Gate 4: Merge
- [ ] `risk:low` + `auto_merge: risk_low_only` → auto-merge into `base_branch`
- [ ] `risk:high` → owner approval (the `risk-high-needs-approval` check)
- [ ] Never automated: merging into a deploy branch (modern-java `master`)

## Multi-repo work
One change per repo. Order with `depends_on` in `agent.yml`. The dispatcher waits until dependencies merge.
Default order for code projects: `db → (jOOQ codegen) → backend → frontend`.

## Stuck Detection Rules
| Condition | Action |
|---|---|
| P-03: same check fails 3× | Comment + `needs-human`, no PR |
| P-04: DoD fails after 2 retries | Comment + `needs-human`, no PR |
| CI fails after 2 fix attempts | `needs-human` |
| P-06: suggestion contradicts a scenario | Conflict comment + `needs-human` |

## Final output
- Living specs updated in the project repo (`openspec/specs/`)
- PR merged into `base_branch`, issue closed
- Run listed in the next daily digest

## Triggers (github connector)
| Event | Starts |
|---|---|
| `/chief <idea>` or an `idea` issue here (owner only) | C-00 |
| `agent-intake` label on a project issue (owner only) | P-01 |
| Push to `base_branch` touching `openspec/changes/**` | spec-sync |
| Cron every 30 min | dispatcher → P-03…P-05 |
| `@claude` from the owner on an `agent/*` PR, or CI failure | P-06 |
| PR merged | P-06 close-out |

Kill switch: repo variable `AGENTS_ENABLED=false`.

## Changelog
- 2026-10-13 — v2.0 — idea entry (C-00), OpenSpec, GitHub connector, risk-based Gate 4
- 2026-06-09 — v1.0 — initial pipeline design (Jira + Bitbucket)
