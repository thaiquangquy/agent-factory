# 06 — Sprint Cadence

## Rhythm

| Cadence | Event | Owner | Your time |
|---|---|---|---|
| Continuous | Agents pick and finish work | Dispatcher | 0 |
| Daily 05:00 | Telegram digest | `daily-digest` workflow | 10–15 min |
| Monday | Content backlog approval (content day) | You + `editor` proposals | 20 min |
| Every 2 weeks, Thursday night | Sprint report generated | `sprint-report` workflow | 0 |
| Every 2 weeks, Friday | **Sprint review + planning** | You | 60–90 min |
| Weekly, Sunday night | Learning PRs | `learning` workflow | Read in the Friday review or the digest |

Start with **2-week sprints**. Change to 1 week only if you have more than 15 merged changes per sprint and decisions wait too long.

## Daily digest (Telegram, 05:00)

Short. You read it in the first minutes of the deep-work block.

```
🏢 Agent Co · Tue 14 Oct

Merged (4): course-java#41 rate-limit-quiz · content#88 short-script-kafka …
Waiting for you (2):
  • hq#12 idea "Quiz analytics" → 2 options ready
  • course-java#43 risk:high PR ready for review
Blocked (1): acme#19 needs-human — spec conflict with auth change
Spend: $6.20 today · sprint $48 / $115
```

Rules:
- Maximum 10 lines. Links on every item.
- Silent days get no message.
- Between 09:00 and 17:00 only `p0` failures send a message.

## Sprint report (generated, `hq/sprints/<id>/report.md`)

The `sprint-reporter` agent writes it from the GitHub Projects board, PRs, CI data, and cost data.

```markdown
# Sprint 2026-S21 (Oct 13 – Oct 24)

## 1. Outcomes against initiatives
| Initiative | Changes merged | Goal progress | Note |

## 2. Shipped (by project)
## 3. Not shipped and why
## 4. Metrics (vs. last sprint, vs. target) — table from file 05
## 5. Incidents: reverts, p0 failures, security events
## 6. Cost: per project vs. budget, cost per merged change
## 7. Learning: rules added, rules proposed for removal, eval trends
## 8. Waiting for your decision (ideas, risk:high PRs, needs-human)
## 9. Proposals for next sprint  ← L4 preparation
| # | Proposed change | Project | Initiative | Why now (evidence) | Effort | Risk |
## 10. Autonomy recommendation per project (stay / up / down, with evidence)
```

## Friday review agenda (60–90 min)

Do it in this order. Stop a section when time runs out, but never skip section 1.

1. **Incidents and reverts (10 min).** Read each one. Decide: new gate, new rule, or risk-policy change.
2. **Metrics (10 min).** Look only at metrics outside the target.
3. **Sample audit (15 min).** Open 2 random auto-merged `risk:low` PRs. Read them as if you were the reviewer. This is how you check that auto-merge is still safe. If you find a problem, that PR type moves to `risk:high`.
4. **Learning PRs (10 min).** Approve, change, or reject.
5. **Vision and initiatives (15 min).** Update `VISION.md` or initiatives if your direction changed.
6. **Planning (15–20 min).** Accept or reject proposals from section 9 in batch. Accepted proposals go to the chief engineer, which writes OpenSpec changes without the 2-option step (you already chose). Set the sprint budget per project.
7. **Autonomy (5 min).** Move projects up or down one level.

Write your decisions in `hq/sprints/<id>/decisions.md`. The agents read this file at the start of the next sprint.

## Moving to L4 (vision → backlog)

At L4 the chief engineer fills section 9 alone, using:
- Initiative goals that are not on track.
- Open bugs and `needs-human` patterns.
- Spec areas with low test coverage.
- Content calendar gaps.

Your job changes from "send ideas" to "approve a backlog in batch on Friday".
Enable L4 per project only after 2 sprints at L3 that meet the exit criteria in file 01.
Keep the 2-option step for any proposal marked `risk:high`.
