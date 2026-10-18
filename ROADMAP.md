# Agent Factory — Roadmap

> Four milestones from a manually-triggered 6-agent pipeline to a fully self-extending,
> self-auditing development organization that runs without engineer intervention.

No timeline is attached. Milestones are sequential — do not start the next one until
the current one meets all pass criteria. Moving fast through Milestone 1 and slow through
Milestone 2 beats rushing to Milestone 4 with a broken foundation.

---

## Overview

```
Milestone 1 — Pipeline runs          6 agents + 7 KB files + manual session spawning
     │
     ▼
Milestone 2 — Pipeline shared        KB reuse across projects + helper agents + capability map
     │
     ▼
Milestone 3 — Meta layer             Orchestrator + Agent-creator + Workspace guardian
     │
     ▼
Milestone 4 — Learning system        Production feedback loop closes back into KB
```

## Milestones ↔ autonomy levels

The autonomy levels come from [`agent-company/01-operating-model.md`](agent-company/01-operating-model.md). Each project's level is in `portfolio.yml`.
Milestones say what the **factory** can do. Levels say what a **project** is allowed to do.

| Milestone | Autonomy it enables | What the owner still does | Rollout phases (agent-company/09) |
|---|---|---|---|
| 1 — Pipeline runs | L2 Task → PR | Writes tickets (`agent-intake`), reviews every PR | 0–2 |
| 2 — Pipeline shared | L2 on a 2nd project | Same | 5 (2nd project) |
| 3 — Meta layer (dispatcher + C-00 + agent-creator + guardian) | **L3 Idea → PR** | Picks options, reviews `risk:high`, sprint review | 3–4 |
| 4 — Learning system (learning agent) | L3 → **L4 Vision → backlog** | Approves the proposed backlog in batch on Friday | 5, 7 |

Exit criteria to move a project up one level: agent-company/01. Never L5.

---

## Milestone 1 — First Pipeline Running

**Goal:** One end-to-end pipeline runs cleanly on real tickets. Human spawns Claude Code sessions manually per phase. The factory produces consistent, spec-driven PRs.

### What to build

**KB layer (9 files):**

| File | Status | Action needed |
|---|---|---|
| `kb/KB-connector-contract.md` | ✅ v1.0 (github) | Verify every `gh` command on the pilot |
| `kb/KB-risk-policy.md` | ✅ v1.0 | Owner review |
| `kb/KB-workflow-conventions.md` | ✅ v2.0 (github) | — (labels created in the pilot) |
| `kb/KB-tech-stack.md` | ✅ modern-java added | — (markdownlint runs on changed `src/` pages in CI) |
| `kb/KB-definition-of-done.md` | ✅ v2.0 | — |
| `kb/KB-context-handoff-contract.md` | ✅ v2.0 | — |
| `kb/KB-tdd-protocol.md` | ✅ v1.1 | — |
| `kb/KB-north-star-principles.md` | ✅ v2.0 | — |
| `kb/KB-repo-map.md` | ⏸ template | Fill in when a multi-repo project joins |

**Agent layer (7 agents, on OpenSpec):**

| Agent | Status |
|---|---|
| `C-00-chief-engineer` | ✅ v1.0 |
| `P-01-ticket-intaker` | ✅ v2.0 |
| `P-02-spec-planner` | ✅ complete |
| `P-03-tdd-implementer` | ✅ complete |
| `P-04-test-verifier` | ✅ complete |
| `P-05-pr-composer` | ✅ complete |
| `P-06-review-resolver` | ✅ complete |

**Infrastructure (you build this):**

- [x] Pilot repo (`thaiquangquy/ModernJava`): `openspec init --tools claude`, OpenSpec 1.14.0 pinned in `package.json`, gates `content-gates.yml` + `openspec-gates.yml` ([thaiquangquy/ModernJava#1](https://github.com/thaiquangquy/ModernJava/pull/1), into `develop`)
- [ ] First living specs via `/opsx:onboard` (owner reviews) and 2 changes by hand (Phase 2 exit)
- [x] Labels from KB-workflow-conventions in the pilot repo (13 labels; `change:<id>` per change)
- [ ] Phase 0 guardrails from agent-company/09: done — kill switch (`AGENTS_ENABLED`), SHA-pinned actions in new workflows, CODEOWNERS on `.github/`, Issues on, `develop` protection (5 required checks, CODEOWNERS review, no force push; admins can bypass). Open — Anthropic Console spend limit (owner); actor check, claude-code-action SHA and `--allowedTools` come with the first agent workflow (Phase 4)
- [ ] Phase 1 control plane gaps: `VISION.md` + initiatives, `company-core` plugin + marketplace file, GitHub Projects board
- [ ] Run the pipeline locally (README → "Running locally") — workflows come in Milestone 3

### Pass criteria

Do not move to Milestone 2 until all three are true:

- [ ] ≥3 real changes ran end-to-end without manual intervention between gates
- [ ] Cycle time per change dropped ≥50% compared to your manual baseline (measure baseline first)
- [ ] OpenSpec changes match what a senior author would write (concrete, checkable scenarios)
- [ ] ≥2 changes merged with updated living specs (`openspec/specs/`)

### Common failure modes at Milestone 1

- **KB files have placeholder values** — nothing works until repo paths and test commands are real
- **Too many hard gates added** — resist adding gates beyond the 4. More gates = pipeline slower than manual
- **Scenarios are vague** — P-03 can only TDD what is checkable. Vague scenarios produce vague checks
- **jOOQ codegen skipped** — if `repo:db` is touched and codegen doesn't run first, backend will fail to compile

---

## Milestone 2 — Pipeline Shared Across Projects

**Goal:** A second project or pipeline type reuses ≥30% of the existing KB and at least one agent. The factory starts generating economies of scale.

### What to build

**Expanded KB:**

- [ ] `kb/KB-capability-map.md` — list the reusable "lego blocks" your system already has (auth, pagination, file upload, notification, etc.). Agents use this to check REUSE before marking NEW
- [x] ~~`kb/KB-project-registry.md`~~ — replaced by `portfolio.yml`
- [ ] Per-project override files: `spec/<project-key>/project.config.md` — test command overrides, custom DoD additions, specific tech stack variations

**New helper agents:**

- [ ] `H-01-context-builder/SKILL.md` — given an OpenSpec change, scans the relevant repos and builds the `relevant_files[]` list for the handoff JSON. Currently P-02 does this manually; extract it as a reusable helper
- [ ] `H-02-jooq-migration-planner/SKILL.md` — specializes in writing Flyway migration SQL and predicting which jOOQ classes will be affected. Called by P-02 when `schema_change: true`

**Naming convention enforcement:**

- [ ] Add `DOC-naming-conventions.md` to `kb/` — the full reference for all ID prefixes, folder structure rules, and deprecation protocol. Milestone 3's guardian will audit against this

### Pass criteria

- [ ] ≥30% of KB files cited by both the original and second pipeline
- [ ] ≥1 helper agent (H-01 or H-02) serving both pipelines
- [ ] `KB-capability-map.md` has ≥5 capabilities defined with REUSE/NEW/EXTEND classification
- [ ] Second project runs ≥2 tickets end-to-end using the shared KB

### Common failure modes at Milestone 2

- **Building pipeline 2 as a full copy of pipeline 1** — this produces two silos with zero KB sharing. The whole point of this milestone is forcing reuse
- **capability-map skipped** — without it, every project invents its own auth, pagination, and notifications. After 3 projects you have 3 incompatible implementations
- **Per-project overrides embedded in SKILL.md** — put project-specific config in `project.config.md`, not in the agent. Agents must stay project-agnostic

---

## Milestone 3 — Meta Layer: Factory Runs Itself

**Goal:** The owner sends an idea and only answers "A" or "B". A cron dispatcher runs the pipeline with no manual session spawning. The owner can describe a new workflow in plain language and the factory creates the agent for it. A guardian keeps the workspace clean. **This is L3.**

### What to build

**Meta-agent 1 — Orchestrator → dispatcher + C-00 (replaces the M-01 webhook design):**

- [x] `agents/C-00-chief-engineer/SKILL.md` — idea → 2 options
- [ ] spec-sync + cron dispatcher workflows from `agent-company/templates/workflows/` (filled in for the pilot)

  Why not a webhook orchestrator: GitHub `concurrency` keeps only one pending run per group, so a label-triggered
  orchestrator loses work. A cron dispatcher picks the top `agent-ready` issue each time (agent-company/03).
  Pipeline state lives in issue labels (`agent-ready`, `agent-running`, `needs-human`), not in a separate store.

  Key behaviors:
  - 1 agent per repo, 3 across the portfolio, budget and daily-run limits from `portfolio.yml` / KB-risk-policy
  - Skips issues whose `depends_on` changes are still open
  - On stuck detection → `needs-human`, never auto-resume
  - Respects the kill switch `AGENTS_ENABLED=false`

**Meta-agent 2 — Agent Creator:**

- [ ] `agents/M-02-agent-creator/SKILL.md`

  Role: when a developer describes a repeating workflow that has no agent, this agent designs and writes the SKILL.md for it. No engineer required.

  Input: plain language workflow description from developer
  Output: complete SKILL.md in the correct `agents/` subfolder, registered in `README-pipeline.md`

  Quality gates the agent-creator applies before writing:
  1. Role fits in 1-2 sentences — if not, split into 2 agents
  2. Input/output are concrete file paths or API calls
  3. SOP has sub-steps with specific verbs and objects
  4. DoD has a checklist, not a feeling
  5. Anti-pattern section exists
  6. Agent does not duplicate an existing agent's role
  7. Agent fits into the existing pipeline order — or is a helper/meta agent

**Meta-agent 3 — Workspace Guardian:**

- [ ] `agents/M-03-workspace-guardian/SKILL.md`

  Role: runs weekly audit of the entire `agents/` and `kb/` directory. Reports drift. Does not auto-fix anything except safe defaults (missing frontmatter fields).

  Checks:
  - All SKILL.md files have required 8 sections
  - All KB files have frontmatter with `name`, `id`, `date`, `status`
  - No IDs changed since last audit (cross-ref integrity)
  - No files deleted (only deprecated)
  - All deprecated files have `pointer` to replacement
  - Cross-references resolve to existing files
  - Naming conventions followed (KB-*, P-*, M-*, H-*, README-*, DOC-*)

  Output: `audit/audit-<YYYY-MM-DD>.md` with issue list + fix recommendations

### Pass criteria

- [ ] ≥3 tickets ran fully automated — developer only acts at hard gates, zero manual session spawning
- [ ] ≥3 new agents created by M-02 in the quarter, triggered by developer description (not engineer)
- [ ] A `risk:low` change goes from idea to merged with the owner's only action being "B"
- [ ] Dispatcher picks the correct issue ≥95% of runs (priority, `depends_on`, budget)
- [ ] Guardian runs weekly and reports drift rate <5% files per week
- [ ] Developer can onboard to the factory in ≤1 day by reading README.md alone

### Common failure modes at Milestone 3

- **Building the dispatcher before the pipeline is stable** — orchestrator built on flaky agents amplifies the flakiness. Milestone 1 and 2 must be solid first
- **Agent-creator with no quality gates** — an agent-creator that just writes whatever it's told produces low-quality agents that silently fail. The 7 filters are non-negotiable
- **Guardian with auto-fix** — a guardian that silently renames files or updates IDs will break cross-references. Report only; humans approve fixes

---

## Milestone 4 — Learning System: KB Stays Fresh

**Goal:** Production feedback flows back into the KB automatically. The factory gets smarter from real usage data, not just from manual KB updates by developers.

### What to build

**Learning agent (design from agent-company/05):** F-01 + F-02 below become one weekly `learning` workflow that opens
reviewed PRs to KB files, skills, and eval cases: ≤ 5 rules per week, each citing its evidence (PRs, reverts, `needs-human` items),
and rules unused for 8 weeks proposed for removal. It never changes gates, the risk policy, budgets, or autonomy levels.
Add promptfoo evals (`evals/chief/` first) so a skill change merges only if its deterministic assertions pass.

**Feedback agent:**

- [ ] `agents/F-01-feedback-listener/SKILL.md`

  Role: after a PR is merged, monitors production signals (error rates, test flakiness, repeated reviewer comments across PRs) and surfaces patterns that suggest a KB entry is stale or a new principle is needed.

  Input: merged PR data + production error logs + aggregated reviewer comment history
  Output: `feedback/feedback-<YYYY-MM-DD>.md` with proposed KB updates — NOT direct KB edits

  Key constraint: F-01 proposes, humans approve. It never writes to KB directly.
  KB is updated by a human or by a designated KB-writer agent after human approval.

- [ ] `agents/F-02-kb-writer/SKILL.md`

  Role: takes an approved feedback proposal from F-01 and writes the KB update with correct frontmatter, ID, changelog entry, and cross-reference updates.

  Trigger: human explicitly approves a proposal from `feedback/` folder

**Supporting infrastructure:**

- [ ] `feedback/` folder added to workspace structure
- [ ] Feedback proposal template: `feedback/FEEDBACK-TEMPLATE.md`
- [ ] KB update review process documented in `DOC-kb-governance.md`

### Pass criteria

- [ ] F-01 running after every merged PR
- [ ] ≥1 KB entry updated per quarter from real production data (not manual authoring)
- [ ] Measurable feedback loop: a repeated reviewer comment pattern leads to a KB principle update within 2 weeks of detection
- [ ] KB entries have `last-validated` date — nothing older than 6 months without a review

### Common failure modes at Milestone 4

- **F-01 writing directly to KB** — this is the highest-risk anti-pattern. Once agents write KB freely, the knowledge layer becomes untrustworthy. Always human-in-the-loop for KB writes
- **Feedback with no signal filtering** — not every reviewer comment is a KB improvement. F-01 must distinguish noise (one-off style preference) from signal (pattern across ≥3 PRs)
- **Skipping F-02 and having F-01 do everything** — separation of concerns: F-01 detects and proposes, F-02 writes. Merging them creates an agent with too broad a scope

---

## Five Signs the Factory Has Reached Full Maturity

From the handbook — measure these to know you're done:

| Sign | How to measure |
|---|---|
| **Onboard fast** | New developer productive in ≤1 day from reading README alone |
| **Resilient to personnel change** | Senior dev leaves — output quality does not drop |
| **Short cycle time** | Feature from ticket assignment to merged PR measured in days, not weeks |
| **Every repeating workflow has an agent** | No developer says "I do this manually every week" |
| **Factory self-extends** | New agents created by M-02 without engineer involvement |

Hitting 5/5 = factory complete. At 3-4/5 = consolidate before adding more agents.

---

## What Never Changes (Across All Milestones)

These constraints are invariants. Any milestone that violates them is going backwards:

- **spec and tests are invariants** — spec deltas are never silently modified during review loops
- **agents are consumers of KB, never editors** — all KB writes go through F-02 or human
- **IDs never change after assignment** — deprecate, never delete, never rename IDs
- **hard gates stay at 4 per pipeline** — more gates means humans become the bottleneck
- **stuck detection always halts** — the bot never retries past its limit and hopes for the best

---

## Current Status

| Milestone | Status | Blocking item |
|---|---|---|
| Milestone 1 | 🟡 In progress | Pilot set up (OpenSpec, labels, gates; 09 Phase 0–2). Next: merge ModernJava#1, console spend limit, `/opsx:onboard` + 2 hand changes, Phase 1 gaps (VISION.md, plugin, board) |
| Milestone 2 | ⬜ Not started | Milestone 1 pass criteria not yet met |
| Milestone 3 | ⬜ Not started | Milestone 2 pass criteria not yet met |
| Milestone 4 | ⬜ Not started | Milestone 3 pass criteria not yet met |

---

*Update the Current Status table as milestones are completed.*
*Last updated: 2026-10-18*
