---
name: KB-risk-policy
id: KB-risk-policy
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: Risk Policy

## Purpose
The risk level decides the review, not habit.
`risk: low` work merges automatically when every blocking gate is green.
`risk: high` work always waits for the owner's approval.

## A change is `risk: high` if ANY item is true

- It changes a public API contract, a DB schema, or a migration. Destructive migrations (DROP, RENAME) are always high (North Star Principle 3).
- It touches auth, security config, payments, PII, or secrets handling.
- It touches more than 1 service or repo, or it changes infrastructure code.
- It changes CI workflows (`.github/`), this policy, hooks, or gates.
- It upgrades a dependency major version, or the pinned OpenSpec / claude-code-action / Claude Code version.
- The project has `auto_merge: never` in `portfolio.yml`.
- Content: it is published under the owner's name to a new channel, or it makes claims about clients, money, or health.
- Content: it changes the book structure (`src/SUMMARY.md` order, chapter removal) rather than adding or editing a page.

Everything else is `risk: low`.

## Who sets it

| Actor | Can |
|---|---|
| C-00 chief-engineer, P-02 spec-planner | Set the level from this policy. Raise it. Never set it below the policy. |
| P-04 test-verifier, P-06 review-resolver, reviewer job | Raise it (add `risk:high`, remove `risk:low`). Never lower it. |
| Owner | Only actor that can lower it. |

## Merge rules

| Situation | Merge |
|---|---|
| `risk: low` and project `auto_merge: risk_low_only` | Auto-merge into the project's `base_branch` when all blocking gates are green |
| `risk: high` | Owner approval required (the `risk-high-needs-approval` check) |
| Project `auto_merge: never` | Owner approval always |
| Merge into a deploy/publish branch (e.g. `master` in modern-java) | Owner only. Never automated. |

## Triggers and permissions (guardrails)

1. Agents start only from: the cron dispatcher, the `idea` label in this repo, or `@claude` from the owner account.
2. Every workflow checks `github.actor` against the owner. Never trust `[bot]` actors.
3. Pin `anthropics/claude-code-action` by commit SHA (≥ v1.0.94), Claude Code ≥ 2.1.128, and exact OpenSpec and promptfoo versions.
4. `permissions:` per job is minimal. Reviewer and verify jobs are read-only plus PR comments.
5. Agent jobs get only `ANTHROPIC_API_KEY` and the GitHub App token. No production secrets.
6. `--allowedTools` is an allow-list per project (in the caller workflow).
7. Hooks deny: `git push --force`, pushes to the base or deploy branch, `rm -rf`, reads of `.env*`, `*.pem`, `*.key`, `~/.ssh`, `/proc/*`, and edits to `.github/workflows/`.
8. Kill switch: repo variable `AGENTS_ENABLED=false` stops every agent workflow.

## Cost limits (start values)

| Control | Value |
|---|---|
| `--max-turns` | 30 (code), 15 (content) |
| Job `timeout-minutes` | 45 |
| Agents per repo at the same time | 1 |
| Agents across the portfolio at the same time | 3 |
| CI fix attempts per PR | 2, then `needs-human` |
| Runs per day per project | 10 |
| Sprint budget | `budget_usd_per_sprint` in `portfolio.yml` |

Source: `agent-company/07-guardrails-security-cost.md`.

## Changelog
- 2026-10-13 — v1.0 — initial version, merged from agent-company/07 and North Star Principle 3
