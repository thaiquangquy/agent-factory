# 02 — Final Stack

Decision date: 2026-10-13
Decision: **Option A (Claude-native stack) + OpenSpec**

## Components

| Layer | Tool | License | Role in the system | Pin |
|---|---|---|---|---|
| Agent runtime | **Claude Code** | Proprietary CLI | Runs every agent, local and in CI | ≥ 2.1.128 (fixes `/proc` secret read) |
| Orchestrator | **`chief-engineer` subagent** (your own) | Yours | Idea → 2 options → OpenSpec change | Versioned in `hq` plugin |
| Spec layer | **OpenSpec** (`@fission-ai/openspec`) | MIT | Proposals, designs, tasks, living specs | Exact version in `package.json`. Upgrade in its own PR. |
| Work queue | **GitHub Issues + GitHub Projects (org board)** | — | Tickets, priority, risk, sprint iteration | — |
| Executor | **anthropics/claude-code-action@v1** | MIT | Runs `/opsx:apply`, opens PRs, fixes CI | Pin by commit SHA, ≥ v1.0.94 |
| CI gates | **GitHub Actions** + language tools | — | Lint, unit tests, build, `openspec validate` | — |
| Evals | **promptfoo** (`anthropic:claude-agent-sdk` provider) | MIT | Skill and prompt regression tests | Exact version |
| Skill tuning | **Anthropic skill-creator** | Apache-2.0 / MIT | Benchmarks skill versions (pass rate, tokens, time) | — |
| Agent memory | **Claude Code subagent memory** (`memory: project`) | — | Learned facts per agent, versioned in git | — |
| Shared config | **Private plugin marketplace** in `hq` repo | Yours | Shared agents, skills, hooks for all projects | Tag releases |
| Notifications | **Telegram** (via a workflow step or Composio) | — | Daily digest, `p0` alerts | — |

## Language tooling per project type

| Project type | Lint / format | Unit tests | Extra gates |
|---|---|---|---|
| Java / Spring Boot | Spotless, Checkstyle | JUnit 5, Testcontainers | ArchUnit (layer rules), JaCoCo threshold |
| React / TypeScript | ESLint, Prettier, `tsc --noEmit` | Vitest | Playwright smoke (optional) |
| Content (markdown) | markdownlint, Vale | Python/Node structure scripts | promptfoo brand-voice rubric |

## Why this stack

- **One investment pays off everywhere.** Skills, hooks, agents, and CLAUDE.md work the same in a local session and in CI.
- **No servers.** Everything runs on GitHub runners. You maintain YAML and markdown, not infrastructure.
- **OpenSpec gives living specs.** Specs describe the current system, so each agent run starts with correct context. This matters more as the portfolio grows.
- **Non-code work fits the same pattern.** An issue produces a markdown PR that goes through gates.

## Known weaknesses and the mitigation

| Weakness | Mitigation |
|---|---|
| claude-code-action had 2 serious security issues in 2026 (bot-actor bypass; secret read through prompt injection). Both are fixed. | Pin SHA ≥ v1.0.94. Pin Claude Code ≥ 2.1.128. Trigger only by label from maintainers or by the dispatcher. See file 07. |
| claude-code-action had a mass-failure wave in Feb 2026. | Pin SHAs. Keep a manual fallback: run the same skill locally. |
| OpenSpec has no native GitHub Issues sync. | Your own `spec-sync.yml` workflow (template provided). |
| OpenSpec `/opsx:archive` skill once did not call the CLI, so specs were dropped silently. | Always run `openspec validate --strict` then the `openspec archive` **CLI**. CI checks that `openspec/specs/` changed. |
| OpenSpec releases often (1.0 in Jan 2026, 1.12 in Sep 2026). Command names change. | Pin the version. Upgrade with `openspec update` in a separate PR per repo. |
| Single vendor (Anthropic). No model fallback. | Accept for now. Keep evals in plain YAML so you can switch later. |
| promptfoo is now owned by OpenAI (still MIT). | Keep eval cases as plain data files. They can move to Inspect or DeepEval. |

## Rejected alternatives (for the record)

| Option | Reason |
|---|---|
| GitHub Agentic Workflows (gh-aw) as the main executor | Best security model, but pre-1.0 with very fast churn. **Revisit** when it reaches 1.0, or use it now for any repo that accepts issues from untrusted people. |
| OpenHands | Needs a server. Claude is no longer its default test model. |
| CCPM | Good issue sync, but no living specs. OpenSpec replaces it. |
| Superpowers brainstorming | Overlaps with `/opsx:explore`. Do not run two planners on the same step. |
| Task Master | Commons Clause license. No native GitHub Issues. |
| Ruflo / claude-flow | Skips permissions by default. Too risky unattended. |
| Archon | Good harness builder. Consider it if GitHub Actions YAML becomes too complex. |
