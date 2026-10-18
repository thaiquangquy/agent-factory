---
name: KB-north-star-principles
id: KB-north-star-principles
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: North Star Principles

## Purpose
These are immutable rules. Agents cite these when making decisions.
They change only when the engineering philosophy changes (yearly cadence).

## Principle 1: The OpenSpec change is the contract
`openspec/changes/<change-id>/` (proposal, design, tasks, spec deltas) is the source of truth for every implementation session.
`openspec/specs/` describes current behavior. Issues only point to the change; they never repeat it.
If the change says X and the issue says Y, the change wins.
If the change is wrong, stop and raise it. Don't implement around it silently.

## Principle 2: Tests are not optional
Every scenario in a spec delta has at least one automated check, named after the scenario.
For code that is a test. For content it is a structure/build check.
No PR is opened with a failing check.

## Principle 3: Schema changes are irreversible — treat them that way
Every migration must be additive where possible.
Destructive migrations (DROP, RENAME column) are always `risk: high` and need the owner's approval.
The jOOQ codegen sequence is non-negotiable.

## Principle 4: Small batches over big bangs
1 commit per scenario. One change = one issue = one implementation PR.
If a change needs more than one PR, split it into more changes with `depends_on`.

## Principle 5: Escalate before guessing
When the agent cannot determine the correct approach with confidence:
do not guess. Comment with the specific blocker, add `needs-human`, and halt.
A wrong assumption that ships is worse than a one-hour delay.

## Principle 6: Spec and tests are invariants during review loops
When a reviewer suggests a change, the agent implements it while keeping the change's scenarios and existing passing tests intact.
If the suggestion contradicts the spec, the agent asks the owner to resolve it. It does NOT silently change the spec.

## Principle 7: Deterministic gates block. LLM judgment advises.
Lint, tests, build, and `openspec validate` can block a merge.
LLM reviews (`/opsx:verify`, reviewer agent, rubric evals) post reports. A `FAIL` can add `needs-human` but never merges or blocks by itself.
If a check matters enough to block, make it deterministic.

## Principle 8: Irreversible actions stay human
Publishing (merging to a deploy branch), deploying to production, contacting anyone, and spending money are done by the owner.
Agents prepare the artifact.

## Principle 9: Risk can be raised, never lowered
Risk follows KB-risk-policy. Any agent may raise it. Only the owner can lower it.

## Changelog
- 2026-10-13 — v2.0 — Principle 1 now names the OpenSpec change; Principles 7–9 added from agent-company
- 2026-06-09 — v1.0 — initial version
