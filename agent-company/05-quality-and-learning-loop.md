# 05 — Quality Gates and the Learning Loop

## Two kinds of checks

| Kind | Examples | Result is stable? | Can block merge? |
|---|---|---|---|
| **Deterministic** | Lint, tests, type check, build, schema, word count, link check, `openspec validate` | Yes | **Yes** |
| **LLM-judged** | `/opsx:verify`, reviewer agent, promptfoo `llm-rubric` | No (varies per run) | **No.** They give a report and trend data. A `FAIL` can add `needs-human`. |

Rule: if a check matters enough to block a merge, make it deterministic.
Example: "The API returns 429 after N requests" becomes an integration test, not an LLM review comment.

## Gate layers

```
Layer 0  Hooks (in the agent session)  — block secret files, block force-push, run formatter after edit
Layer 1  Agent self-check              — agent runs lint + tests before it opens the PR
Layer 2  CI deterministic gates        — lint, tests, coverage, build, openspec validate, archive check
Layer 3  CI LLM advisors               — /opsx:verify, reviewer agent
Layer 4  Risk policy                   — risk:high → human review
Layer 5  Sprint review                 — human looks at trends and samples
```

Each layer catches different failures. Do not remove a layer because a later layer exists.

## Spec scenarios become tests

OpenSpec scenarios use GIVEN / WHEN / THEN.
Add this line to every project's `CLAUDE.md`:

> Every scenario in a spec delta must have at least one automated test. Name the test after the scenario.

Then `/opsx:verify` can check traceability: scenario → test name → test result.
This turns the spec into a test oracle. The agent cannot pass by writing code that only "looks right".

## Eval harness

Evals test the **agents and skills**, not the product code.

| Suite | Location | What it tests | When it runs |
|---|---|---|---|
| Skill regression | `hq/evals/skills/` | Each skill on fixed input cases (e.g., "write a Spring controller with validation") | PR that changes a skill |
| Chief-engineer quality | `hq/evals/chief/` | 10–20 past ideas. Checks: exactly 2 options, effort/risk present, a recommendation present, no file writes before approval | PR that changes `chief-engineer.md` |
| Brand voice | `hq/evals/content/` | Content drafts against the brand rubric | PR that changes `editor.md` or brand skills |
| PR description quality | `hq/evals/pr/` | PR body has: change, tests, scenarios, risks | Weekly |

promptfoo setup (sketch):

```yaml
# hq/evals/chief/promptfooconfig.yaml
providers:
  - id: anthropic:claude-agent-sdk
    config:
      working_dir: ./fixtures/course-java
      append_allowed_tools: [Read, Grep, Glob]
prompts:
  - "/chief {{idea}}"
tests:
  - vars: { idea: "Add rate limiting to the quiz endpoint" }
    assert:
      - type: javascript            # deterministic — blocks
        value: "(output.match(/### Option [AB]/g) || []).length === 2"
      - type: contains              # deterministic — blocks
        value: "### Recommendation"
      - type: llm-rubric            # advisory — trend only
        value: "The two options differ in approach, not only in detail."
```

Run each LLM-graded case **3 times** and record mean ± standard deviation.
A skill change merges only if deterministic assertions pass and the rubric mean does not drop more than 1 standard deviation.

## The learning loop

The system improves through **reviewed PRs**, never through silent self-edits.

```
Signals ─► learning agent (weekly) ─► rule/skill PR ─► eval before/after ─► you approve ─► merged
```

### Signals the learning agent reads (last 7 days, all repos)

1. Your review comments on PRs (`risk:high` reviews especially).
2. CI failures that needed fix attempts, grouped by cause.
3. `needs-human` items and how you resolved them.
4. Reverted PRs.
5. `/opsx:verify` FAIL reports.
6. Your edits to agent-written content before publishing (diff of draft vs. final).

### What it can propose

| Target | Example |
|---|---|
| Project `CLAUDE.md` → "Learned rules" | "Use `@Transactional(readOnly = true)` on query services." |
| A shared skill in `hq` | Add a section about Testcontainers reuse to `spring-boot-conventions` |
| Subagent memory (`.claude/agent-memory/`) | Facts about module boundaries |
| A new eval case | The case that failed last week becomes a regression test |
| `risk-policy.md` | "Changes to `SecurityConfig` are always risk:high" |

### Rules for learned rules

- Each rule cites its evidence (PR or issue links).
- Maximum 5 proposed rules per week. Quality over volume.
- A rule that no failure has triggered for 8 weeks goes to the sprint report as "candidate for removal". Rules files grow and degrade context if nobody prunes them.
- The learning agent cannot change gates, the risk policy, budgets, or autonomy levels without your approval in the sprint review.

## Metrics (tracked per project, per sprint)

| Metric | Formula | L3 target |
|---|---|---|
| First-pass gate rate | PRs green on first CI run / all PRs | ≥ 70% |
| Final gate rate | PRs green within 2 fix attempts / all PRs | ≥ 90% |
| Auto-merge revert rate | Reverted auto-merged PRs / auto-merged PRs | 0% |
| Human touch rate | PRs with `needs-human` / all PRs | ≤ 15% |
| Rework rate | `risk:high` PRs with "request changes" / `risk:high` PRs | ≤ 30% |
| Lead time | Option chosen → merged | Trend down |
| Cost per merged change | USD / merged PR | Inside budget |
| Spec coverage | Scenarios with a linked test / all scenarios | 100% |

The sprint reporter computes these. File 06 shows how you use them.
