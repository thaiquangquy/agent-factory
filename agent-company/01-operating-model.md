# 01 — Operating Model

## The company metaphor

Think of the system as a small company.

| Company role | Who | Main output |
|---|---|---|
| CEO / founder | **Quy (human)** | Vision, initiatives, option choices, sprint decisions |
| Chief engineer | `chief-engineer` agent (Claude Code, local or via issue) | 2 options per idea, OpenSpec changes, backlog proposals |
| Implementer | `claude-code-action` in each project repo | Implementation PRs |
| Reviewer | `reviewer` agent (read-only CI job) | PR review comments, risk check |
| Verifier | `/opsx:verify` CI job | Spec-vs-code report |
| Editor | `editor` agent (content repos) | Content PRs that follow brand rules |
| Retro facilitator | `learning` agent (weekly scheduled run) | PRs that change skills and rules |
| Chief of staff | `sprint-reporter` agent (scheduled run) | Daily digest, sprint report |

The human does not write tickets, assign work, or review low-risk code.
The human does decide direction, approve options, review high-risk work, and run the sprint review.

## Decision rights

| Decision | Human | Agent |
|---|---|---|
| Vision, goals, which projects exist | **Decides** | Proposes changes in sprint report |
| Which option to build (A or B) | **Decides** | Proposes 2 options + recommendation |
| Spec content (proposal, design, scenarios) | Approves through option choice; reviews if `risk:high` | **Writes** |
| Task breakdown and priority inside a change | Can override | **Decides** |
| Implementation details | — | **Decides** |
| Merge `risk:low` PR | Optional review | **Auto-merges** when all gates are green |
| Merge `risk:high` PR | **Decides** | Prepares |
| Change a skill, rule, or eval | **Approves PR** | Proposes PR |
| Publish content, deploy to prod, contact a client, spend money | **Does it** | Prepares the artifact |
| Raise risk level of a change | — | **Can raise** (never lower) |

## Autonomy levels

Each project has an autonomy level in `hq/portfolio.yml`.
A project moves up one level only after a sprint review where it meets the exit criteria.

| Level | Name | What the agent does alone | What the human does | Exit criteria to next level |
|---|---|---|---|---|
| L1 | Assistant | Writes code when asked, in a local session | Everything else | — |
| L2 | Task → PR | Picks a ticket, opens a PR | Writes tickets, reviews every PR | 2 sprints with ≥ 80% PRs merged without major rework |
| **L3** | **Idea → PR (target now)** | Explores, gives options, writes OpenSpec change, creates issue, implements, auto-merges `risk:low` | Picks options, reviews `risk:high`, runs sprint review | 2 sprints with: 0 reverted auto-merges, ≥ 90% gate pass rate, cost inside budget |
| L4 | Vision → backlog | Also proposes new changes from vision, metrics, and bug reports at sprint planning | Approves or rejects the proposed backlog in batch | Decide later |
| L5 | Self-directed | Starts work without human approval | Audits | **Not recommended.** Keep a human gate on intent. |

You are at **L2** now. These files move you to **L3**, then prepare for **L4**.

## Work hierarchy

```
Vision (hq/VISION.md)                     — human, changes rarely
 └─ Initiative (hq/initiatives/*.md)      — human + chief engineer, 2–8 weeks
     └─ OpenSpec change (project repo)    — chief engineer, 1–5 days of agent work
         └─ GitHub Issue                  — created by workflow, 1 per change
             └─ Implementation PR         — implementer agent
```

Rules:
- Every change links to one initiative. A change with no initiative is "maintenance" and needs a reason.
- One change produces one issue and one implementation PR. If a change needs more than one PR, the chief engineer splits it into more changes.
- Issues do not repeat the spec. They link to the change folder.

## Human time budget

The system respects your weekly schedule.

| When | What | Time |
|---|---|---|
| Daily, 05:00–05:15 (deep-work block) | Read the Telegram digest. Pick options for pending ideas. Unblock `needs-human` items. | 15 min |
| Any time | Send new ideas (laptop `/chief` or phone `idea` issue) | As needed |
| Monday (content day) | Approve the content backlog for the week | 20 min |
| Friday (review and planning) | Sprint review and planning, every 1–2 weeks | 60–90 min |

No agent notifications go out during your 09:00–17:00 corporate block, except `p0` failures.

## What stays outside the system

- **Employer work** (Personify, Thoughtworks clients). Do not connect employer repos, tickets, or secrets to this system. Keep IP and access separate.
- **Client communication.** Agents draft. You send.
- **Money.** Agents never buy, invoice, or transfer.
