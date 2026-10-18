---
name: KB-workflow-conventions
id: KB-workflow-conventions
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: Workflow Conventions

Connector: `github` (see KB-connector-contract). Bitbucket/Jira conventions are at the end, for later.

## Work states (labels)

```
idea ──C-00──► (option chosen) ──P-02──► spec PR merged ──spec-sync──► agent-ready
                                                                          │ dispatcher
                                                                          ▼
                     needs-human ◄── stuck / 2 CI fails ── agent-running ──P-05──► PR open ──merge──► closed
```

| Label | Meaning | Set by |
|---|---|---|
| `idea` | Raw idea for C-00 (phone entry, in this repo) | Owner |
| `agent-intake` | Ticket written by the owner; P-01 validates it | Owner |
| `agent-ready` | Dispatcher can pick it | spec-sync |
| `agent-running` | An agent works on it now (max 1 per repo) | Dispatcher |
| `needs-human` | Agent stopped. Owner input needed. | Any agent or CI |
| `priority:p0` … `priority:p3` | Priority | spec-sync (from `agent.yml`) |
| `risk:low`, `risk:high` | Review policy (KB-risk-policy) | spec-sync; can only be raised by agents |
| `change:<change-id>` | Links issue and PR to an OpenSpec change | spec-sync, P-05 |
| `type:code`, `type:content` | Project type | spec-sync |

## Comment format

Agent comments on issues and PRs always use:

```
[AGENT] <phase>

<content>

---
Change: openspec/changes/<change-id>/
Agent: <C-00|P-01..P-06> | <ISO timestamp>
```

## Branches

| Branch | Created by | Example |
|---|---|---|
| `spec/<change-id>` | P-02 (spec PR) | `spec/add-records-chapter` |
| `agent/<change-id>` | P-03 (implementation PR) | `agent/add-records-chapter` |

Base branch = `base_branch` in `portfolio.yml` (`develop` for modern-java).
The `openspec-gates` archive check runs only on `agent/*` heads.

## Change id
kebab-case, verb first, ≤ 40 chars: `add-records-chapter`, `fix-switch-typos`.

## PR titles
- Spec PR: `[spec] <change-id>: <summary>`
- Implementation PR: `[<change-id>] <summary>`

## Implementation PR body template

```markdown
Closes #<issue>

## Summary
<2-3 sentences from proposal.md>

## Scenarios
- [x] <scenario name> — <test or check name>

## Spec
Change: openspec/changes/archive/<date>-<change-id>/
Living specs updated: openspec/specs/<capability>/spec.md

## Verification
| Gate | Result |
|---|---|
| <gate> | PASS |

## Risk
risk:<low|high> — <policy rule that applies, or "no rule matched">

## Open risks
<none | list>
```

## Issue body (created by spec-sync)

```markdown
Change: openspec/changes/<change-id>/
Initiative: <id or maintenance>
Acceptance: all scenarios in the spec deltas pass; tasks.md is fully checked.
Do not copy the spec here. Read the change folder.
```

## Future connector: bitbucket + Jira (not implemented)

Kept from v1.0 for when the `bitbucket` connector is built.

| Generic state | Jira status |
|---|---|
| `agent-intake` | Backlog + label `dev-bot` + assignee = owner |
| `agent-ready` | planning → IN PROGRESS (owner moves it) |
| `agent-running` | IN PROGRESS |
| PR open | review |
| closed | done |
| `needs-human` | blocked (+ label `bot-blocked` / `needs-clarification`) |

Bitbucket events to map: `pullrequest:comment_created` → P-06, `pullrequest:fulfilled` → P-06 close-out, `pullrequest:rejected` → comment + back to planning.
Branch naming on Bitbucket keeps the same `agent/<change-id>` rule.

## Changelog
- 2026-10-13 — v2.0 — GitHub labels/branches from agent-company/03; Jira/Bitbucket moved to the future-connector section
- 2026-06-09 — v1.0 — initial version (Jira + Bitbucket)
