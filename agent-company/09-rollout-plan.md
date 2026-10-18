# 09 — Rollout Plan: L2 → L3 → L4

You already have a task → PR workflow (L2).
This plan adds one capability at a time. Each phase must work for 1 week before the next phase starts.
Start with **one pilot project** (a code project you own, not client work). Add other projects in phase 5.

## Phase 0 — Audit the current workflow (1 session)

Check the current task → PR workflow against this list. Fix the gaps before phase 1.

- [ ] claude-code-action is pinned by SHA, at v1.0.94 or later
- [ ] Claude Code is 2.1.128 or later
- [ ] Triggers check the actor (your account only, no `[bot]` actors)
- [ ] `permissions:` is minimal per job
- [ ] `--max-turns` and `timeout-minutes` are set
- [ ] `--allowedTools` is an allow-list
- [ ] Agents cannot edit `.github/workflows/` (CODEOWNERS + branch protection)
- [ ] Lint and unit tests run on every agent PR, and they block merge
- [ ] One Anthropic Console workspace with a spend limit exists for agent runs
- [ ] A kill switch variable exists

**Done when:** every box is checked.

## Phase 1 — Control plane (1 week)

- [ ] Create the `hq` repo with the layout in file 03
- [ ] Write `VISION.md` (1 page) and 1–2 initiatives
- [ ] Write `portfolio.yml` with the pilot project only
- [ ] Write `rules/risk-policy.md` from file 07
- [ ] Create the `company-core` plugin and the marketplace file
- [ ] Move existing shared skills into the plugin
- [ ] Create the org GitHub Projects board with the fields from file 03
- [ ] Create the labels in the pilot repo

**Done when:** a local Claude Code session in the pilot repo loads the plugin.

## Phase 2 — OpenSpec in the pilot repo (1 week)

- [ ] `npm i -D @fission-ai/openspec@<exact>` and `openspec init --tools claude`
- [ ] Run `/opsx:onboard` to create first specs for the 2–3 most important areas. Review them yourself — this is the foundation.
- [ ] Add the "every scenario has a test" rule to `CLAUDE.md`
- [ ] Add CI jobs: `openspec validate --strict`, archive check
- [ ] Do 2 changes by hand with `/opsx:propose` → `/opsx:apply` → CLI archive, locally

**Done when:** 2 changes merged with updated living specs.

## Phase 3 — Chief engineer + spec-sync (1 week)

- [ ] Add `chief-engineer.md` and `/chief` (templates folder)
- [ ] Add `spec-sync` reusable workflow + caller in the pilot repo
- [ ] Add `agent.yml` to the change template
- [ ] Test: 3 ideas through `/chief` → options → spec PR → issue

**Done when:** 3 issues were created from ideas, with correct labels and board fields, and you wrote no ticket by hand.

## Phase 4 — Dispatcher, verify, risk-based merge (1–2 weeks)

- [ ] Replace your current task trigger with the cron dispatcher
- [ ] Update the agent-run prompt to `/opsx:apply` + CLI archive (file 04, step 7)
- [ ] Add `reusable-verify.yml` (advisory) and the reviewer job
- [ ] Add the CI fix loop (max 2 attempts → `needs-human`)
- [ ] Turn on GitHub auto-merge. Branch protection: required checks for all PRs, required review for `risk:high` (use a ruleset or a required check that fails when `risk:high` has no approval)
- [ ] Add `daily-digest` to Telegram

**Done when:** a `risk:low` change goes from idea to merged with your only action being "B".
**This is L3.**

## Phase 5 — Learning loop and sprint review (2 weeks = first sprint)

- [ ] Add the `hq/evals/chief/` suite with 10 past ideas
- [ ] Add `weekly-learning` and `sprint-report` workflows
- [ ] Run the first Friday sprint review with the agenda in file 06
- [ ] Add the content repo (file 08) at L3
- [ ] Add a client project at L2 (`auto_merge: never`)

**Done when:** one full sprint review is complete, and you have merged at least 1 learning PR.

## Phase 6 — Phone entry and multi-project (1 week)

- [ ] Create the GitHub App with access to listed repos only
- [ ] Add `idea.yml` in `hq`
- [ ] Dispatcher reads `portfolio.yml` for budgets and the cross-project limit

**Done when:** an idea from your phone becomes a merged PR.

## Phase 7 — L4 (after 2 good sprints at L3)

- [ ] Turn on section 9 (proposals) in the sprint report for qualified projects
- [ ] Accepted proposals go to the chief engineer without the 2-option step (except `risk:high`)
- [ ] Track: proposal acceptance rate. Target ≥ 60%. Lower means the chief engineer does not understand the vision yet — improve `VISION.md` and initiatives, not the agent.

## Timeline

| Week | Phase |
|---|---|
| 1 | 0 + 1 |
| 2 | 2 |
| 3 | 3 |
| 4–5 | 4 → **L3 reached** |
| 6–7 | 5 (first sprint) |
| 8 | 6 |
| 10+ | 7 → **L4 for qualified projects** |

Do the build work in the 05:00–07:00 deep-work blocks. Each phase is about 4–6 blocks.

## What to measure from the start

Start collecting these in phase 0, so you have a baseline for the L3 comparison:
- Your hours per week on this system (target: < 3 h at L3, excluding the sprint review)
- Merged changes per week
- Cost per merged change
- Revert count
