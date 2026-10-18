# 04 — Main Use Case: Idea → 2 Options → OpenSpec → PR

## Summary

> I have an idea. I type it to the chief engineer agent. It gives me 2 options with pros and cons.
> If I agree, the task is created with OpenSpec, and Claude Action handles the ticket.
> I verify the final result when the risk needs it.

## Actors

- **Quy**: gives the idea, chooses the option.
- **chief-engineer**: explores, proposes, writes the OpenSpec change.
- **spec-sync**: creates the issue.
- **dispatcher + agent-run**: implements.
- **CI, verify, reviewer**: check the work.

## Entry points

| Entry | When | How | Speed |
|---|---|---|---|
| **Laptop (primary)** | Deep-work block, detailed ideas | `claude` in the `hq` folder, then `/chief <idea>` | Fast, interactive |
| **Phone** | Ideas during the day | New issue in `hq` with the `idea` label | 1–2 min per turn |

Both entries use the same `chief-engineer.md` agent file.

## Step-by-step

### Step 1 — Idea

```
/chief Add rate limiting to the course API. Free users abuse the quiz endpoint.
```

### Step 2 — Explore (agent, no writes)

The chief engineer:
1. Reads `hq/VISION.md`, `portfolio.yml`, and the related initiative.
2. Finds the target repo. If it is not clear, it asks you one question.
3. Runs `/opsx:explore` in the target repo. It reads `openspec/specs/` and the related code.
4. Checks for open changes in the same area to prevent conflicts.

### Step 3 — Two options (agent → you)

The answer always has this format:

```markdown
## Idea: Rate limit the quiz endpoint
Target: course-java · Initiative: 2026-Q4-platform-hardening

### Option A — Bucket4j in the app, Redis backend
- Approach: …
- Pros: …
- Cons: …
- Effort: M (≈ 1 agent run) · Risk: high (public API behavior) · Services: course-api

### Option B — Rate limit at Spring Cloud Gateway
- …

### Recommendation: B, because …
### Open questions (max 2): …

Reply "A", "B", "B with <change>", or "drop".
```

Rules for the agent:
- Exactly 2 options. They must differ in approach, not only in detail.
- Each option states effort, risk, and affected services.
- The agent must recommend one and say why. "It depends" is not allowed.
- The agent stops here. It writes no files before your answer.

### Step 4 — You choose

```
B, but keep the limit configurable per plan.
```

### Step 5 — OpenSpec change (agent)

1. Runs `/opsx:propose rate-limit-quiz` in the target repo with option B and your note.
2. Writes `agent.yml` (initiative, priority, risk). The agent sets risk with `hq/rules/risk-policy.md`. It can raise risk but never lower it below the policy.
3. Runs `openspec validate rate-limit-quiz --strict`. It fixes errors.
4. Shows you a 5-line summary: scope, number of tasks, scenarios, risk, priority.
5. Creates branch `spec/rate-limit-quiz`, commits, and opens a **spec PR**.
   - `risk:low` → auto-merge is on.
   - `risk:high` → auto-merge is on too. Your option choice already approved the intent. Read the PR only if you want to.

### Step 6 — Ticket (workflow)

When the spec PR merges, `spec-sync` runs:
1. Finds new folders under `openspec/changes/` (not `archive/`).
2. Validates each one.
3. Creates one issue if one does not exist already (idempotent check on the `change:<id>` label).

Issue body:

```markdown
Change: openspec/changes/rate-limit-quiz/
Initiative: 2026-Q4-platform-hardening
Acceptance: all scenarios in the spec deltas pass; tasks.md is fully checked.
Do not copy the spec here. Read the change folder.
```

Labels: `agent-ready`, `priority:p1`, `risk:high`, `change:rate-limit-quiz`.

### Step 7 — Implementation (dispatcher + claude-code-action)

The dispatcher runs every 30 minutes:
1. Stops if this repo already has an `agent-running` issue (1 agent per repo).
2. Stops if the project's sprint budget is used up.
3. Picks the top `agent-ready` issue: priority first, then oldest.
4. Sets `agent-running`, then calls claude-code-action with:

```
Implement OpenSpec change rate-limit-quiz.
1. Run /opsx:apply rate-limit-quiz. Complete every task in tasks.md.
2. Run the project lint and unit tests. Fix failures.
3. Run: openspec validate rate-limit-quiz --strict
4. Run the CLI: openspec archive rate-limit-quiz --yes
   (Do not use the archive skill. Use the CLI.)
5. Open a PR. Title: "[rate-limit-quiz] <summary>". Body: "Closes #<issue>".
   Include: what changed, test results, scenarios covered, open risks.
```

**Archive is inside the implementation PR.** Code and the living spec change in the same merge. If the PR is reverted, the spec is reverted too.

### Step 8 — Gates

| Gate | Blocks merge? |
|---|---|
| Lint, format, type check | Yes |
| Unit tests + coverage threshold | Yes |
| `openspec validate --strict` on all specs | Yes |
| Archive check: change folder moved to `archive/` and `openspec/specs/` changed | Yes |
| Build | Yes |
| `/opsx:verify` (LLM: code vs. scenarios) | No — posts a report. A `FAIL` adds `needs-human`. |
| Reviewer agent (LLM review) | No — posts comments |
| promptfoo evals (if prompts or skills changed) | Only deterministic assertions block |

If a blocking gate fails, Claude gets **2 fix attempts** (`@claude fix the failing CI checks`).
After 2 failures: label `needs-human`, remove `agent-running`, and add the item to the daily digest.

### Step 9 — Merge and report

| Risk | Merge |
|---|---|
| `risk:low` | GitHub auto-merge when all blocking gates are green |
| `risk:high` | Branch protection needs your approval |
| Project with `auto_merge: never` | Always your approval |

After merge: the issue closes, the board moves to Done, and the run cost goes into the board's Cost field.
The next 05:00 digest lists it.

## Failure paths

| Situation | System behavior |
|---|---|
| You do not answer the options for 7 days | The idea goes to the sprint report under "waiting for decision". |
| The idea crosses 2 repos | The chief engineer makes 2 changes, 1 per repo, and lists the order. The dispatcher respects `depends_on` in `agent.yml`. |
| The spec is wrong (found during implementation) | The agent stops, comments on the issue, and sets `needs-human`. It does not change the spec intent alone. |
| CI fails 2 times | `needs-human`, then the digest. |
| A merged PR causes a bug | Revert PR. The `learning` agent reads the revert in its weekly run. |
| Budget is used up | The dispatcher stops for that project and reports it in the digest. |

## Phone version (idea issue)

1. You open an issue in `hq`: title = idea, label = `idea`.
2. `idea.yml` runs claude-code-action with the `chief-engineer` agent. It posts the 2 options as a comment.
3. You reply `@claude B` from the GitHub mobile app.
4. The agent continues from step 5. It opens the spec PR in the target repo with a cross-repo token (see file 07).
