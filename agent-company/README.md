# Agent Company — Operating Blueprint

Version: 1.0 (2026-10-13)
Owner: Quy

## Purpose

This set of files defines how one human runs a portfolio of projects with AI agents.
The portfolio has coding projects (agency work, products) and non-code projects (content creation).

The human gives the vision and makes high-level decisions.
Agents plan, build, test, and report.
The human reviews the results in a sprint review every 1–2 weeks.

## Read in this order

| # | File | What it gives you |
|---|------|-------------------|
| 01 | [operating-model.md](01-operating-model.md) | Roles, decision rights, autonomy levels |
| 02 | [final-stack.md](02-final-stack.md) | The selected tools, versions, and the reasons |
| 03 | [architecture.md](03-architecture.md) | Repo layout, control plane, data flow |
| 04 | [use-case-idea-to-pr.md](04-use-case-idea-to-pr.md) | The main flow: idea → 2 options → OpenSpec → PR |
| 05 | [quality-and-learning-loop.md](05-quality-and-learning-loop.md) | Gates, evals, and how results improve over time |
| 06 | [sprint-cadence.md](06-sprint-cadence.md) | Daily digest, sprint review, sprint planning |
| 07 | [guardrails-security-cost.md](07-guardrails-security-cost.md) | Security, cost limits, failure handling |
| 08 | [content-projects.md](08-content-projects.md) | How non-code work uses the same system |
| 09 | [rollout-plan.md](09-rollout-plan.md) | Steps from the current task→PR workflow to the next autonomy level |
| — | [templates/](templates/README.md) | Agent file, `/chief` command, workflow skeletons, gotchas |

## One-paragraph summary

You type an idea to the **chief engineer** agent.
It explores the code and the living specs, and it gives you 2 options with pros and cons.
You choose one.
The chief engineer writes an **OpenSpec change** in the target repo and opens a spec PR.
When the spec PR merges, a workflow creates a **GitHub Issue** with priority and risk labels.
A **dispatcher** gives the issue to **claude-code-action**, which runs `/opsx:apply` and opens an implementation PR.
CI runs lint, unit tests, OpenSpec validation, and evals.
Low-risk PRs merge automatically. High-risk PRs wait for you.
Every 1–2 weeks, an agent writes a sprint report, and you review it on Friday.

## Core principles

1. **Human owns intent. Agents own execution.** You approve *what* and *why*. Agents decide *how*, inside guardrails.
2. **Specs are the source of truth.** OpenSpec specs describe current behavior. Issues are only work tickets that point to specs.
3. **Deterministic gates block. LLM judgment advises.** Lint, tests, and schema checks can block a merge. LLM-graded evals show trends.
4. **Every learned rule is a reviewed PR.** The system improves through versioned changes to skills, CLAUDE.md, and agent memory. Nothing changes silently.
5. **Risk decides the review, not habit.** Low-risk work auto-merges. High-risk work always gets a human review.
6. **Irreversible actions stay human.** Publishing content, deploying to production, sending client email, and spending money need a human action.
