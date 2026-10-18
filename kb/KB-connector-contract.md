---
name: KB-connector-contract
id: KB-connector-contract
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: Connector Contract

## Purpose
Agents never call a Git host or tracker API by name. They call one of the operations below.
Each project's `connector` in `portfolio.yml` decides which column applies.

The contract stays small on purpose: only the operations an agent SOP really calls.
Add an operation only when a SKILL.md needs it, and in the same PR.

## Connectors

| Connector | Tracker | Code host | Runner | Status |
|---|---|---|---|---|
| `github` | GitHub Issues | GitHub | `anthropics/claude-code-action` (CI), Claude Code (local) | **active** |
| `bitbucket` | Jira | Bitbucket | Local Node daemon (`claude -p`) | **not implemented** |

## Operations

`<repo>` is the project's `repo` from `portfolio.yml`. Always pass it explicitly (`--repo`).
Never rely on the git remote: a fork has an `upstream` remote, and agents must never target it.

| # | Operation | Used by | `github` (gh CLI) | `bitbucket` |
|---|---|---|---|---|
| 1 | `create_issue(title, body, labels[])` | spec-sync, P-01 | `gh issue create --repo <repo> --title <t> --body-file <f> --label <l1>,<l2>` | not implemented |
| 2 | `list_issues(labels[], state)` | dispatcher, C-00 | `gh issue list --repo <repo> --label <l> --state <s> --json number,title,labels,createdAt` | not implemented |
| 3 | `comment(issue_or_pr, body)` | all agents | `gh issue comment <n> --repo <repo> --body-file <f>` (PRs: `gh pr comment`) | not implemented |
| 4 | `set_labels(issue_or_pr, add[], remove[])` | dispatcher, P-01, P-04, P-06 | `gh issue edit <n> --repo <repo> --add-label <a> --remove-label <r>` (PRs: `gh pr edit`) | not implemented |
| 5 | `close_issue(issue, reason)` | P-06 | `gh issue close <n> --repo <repo> --reason <completed\|not planned>` | not implemented |
| 6 | `open_pr(base, head, title, body, labels[])` | P-02, P-05 | `gh pr create --repo <repo> --base <base> --head <head> --title <t> --body-file <f> --label <l>` | not implemented |
| 7 | `read_review_comments(pr)` | P-06 | `gh api repos/<repo>/pulls/<n>/comments` + `gh pr view <n> --repo <repo> --json reviews,comments` | not implemented |
| 8 | `get_checks(pr)` | P-04, P-06 | `gh pr checks <n> --repo <repo> --json name,state,bucket` | not implemented |
| 9 | `enable_auto_merge(pr)` | P-02, P-05 (only when `risk: low` and `auto_merge` allows it) | `gh pr merge <n> --repo <repo> --auto --squash` | not implemented |
| 10 | `get_pr_state(pr)` | P-06 | `gh pr view <n> --repo <repo> --json state,mergedAt,url,labels` | not implemented |

Branch creation, commits, and pushes use plain `git`, so they are not part of the contract.

## Events (what starts an agent)

| Event | `github` source | Agent |
|---|---|---|
| New idea | `/chief <idea>` locally, or an issue labelled `idea` in this repo | C-00 |
| Ticket written by a human | Issue labelled `agent-intake` in the project repo | P-01 |
| Spec PR merged | `push` to the base branch on `openspec/changes/**` → spec-sync | (creates issue) |
| Issue ready | Cron dispatcher, every 30 min, picks the top `agent-ready` issue | P-03 → P-04 → P-05 |
| Review comment on an agent PR | `@claude` from the owner on a PR with head `agent/*` | P-06 |
| PR merged | `pull_request: closed` with `merged == true` | P-06 (close-out) |

Every event checks the actor first: only the owner account. Never trust actors that end in `[bot]` (see KB-risk-policy).

## Rules
- An agent that needs an operation not in this table stops and posts `CONNECTOR_GAP: <operation>` to the issue. It does not call an API directly.
- When `connector` is `bitbucket`, every agent stops with `CONNECTOR_NOT_IMPLEMENTED: bitbucket`.

## Changelog
- 2026-10-13 — v1.0 — initial version, GitHub only
