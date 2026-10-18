# Templates

These are **skeletons**. They show the structure and the important guards.
Read every line, replace every `<...>` value, and test in the pilot repo first.

| File | Copy to | Purpose |
|---|---|---|
| `plugin/agents/chief-engineer.md` | `hq/plugins/company-core/agents/` | Orchestrator: idea → 2 options → OpenSpec change |
| `plugin/commands/chief.md` | `hq/plugins/company-core/commands/` | `/chief <idea>` entry point |
| `workflows/reusable-spec-sync.yml` | `hq/.github/workflows/` | Change merged → issue |
| `workflows/reusable-dispatch-and-run.yml` | `hq/.github/workflows/` | Pick top issue → claude-code-action → PR |
| `workflows/agents-caller.yml` | `project-x/.github/workflows/agents.yml` | Thin caller per project |
| `workflows/openspec-gates.yml` | `project-x/.github/workflows/` | Blocking OpenSpec gates + risk:high approval check |
| `agent.yml.example` | OpenSpec change template | Metadata per change |

## Not included yet (build in the phase shown in file 09)

| Workflow | Phase |
|---|---|
| `ci-fix` loop (max 2 attempts → `needs-human`) | 4 |
| `reusable-verify.yml` + reviewer job (advisory) | 4 |
| PR-opened → remove `agent-running` | 4 |
| `daily-digest.yml` (Telegram) | 4 |
| `weekly-learning.yml`, `sprint-report.yml` | 5 |
| `idea.yml` (phone entry in `hq`) | 6 |

## Gotchas you will hit

1. **PRs and issues created with `GITHUB_TOKEN` do not trigger other workflows.** Your CI will not run on the agent PR. Use the GitHub App token (already in the skeletons).
2. **Reusable workflows from a private `hq` repo.** In `hq` → Settings → Actions → General → Access, allow access from your other repos. Otherwise callers fail with "workflow not found".
3. **`secrets: inherit`** passes the caller's secrets. Each project repo needs `ANTHROPIC_API_KEY`, `APP_ID`, `APP_PRIVATE_KEY`. Use one Anthropic workspace key per project to get cost per project.
4. **GitHub `concurrency`** keeps only 1 pending run. This is why the dispatcher is cron-based, not label-triggered.
5. **Cron on GitHub is best-effort.** Runs can be 5–15 minutes late under load. Do not depend on exact times.
6. **`openspec` command names change between versions.** Check `openspec --help` for your pinned version, and update the prompts in the same PR as the version bump.
7. **claude-code-action inputs** change between v1 releases. Check the action's `action.yml` at your pinned SHA.
