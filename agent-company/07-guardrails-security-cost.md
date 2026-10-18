# 07 — Guardrails: Security, Cost, Failure

## Threat model (short)

The main risk is **untrusted text that reaches an agent that has write access**.
Untrusted text includes: issue bodies, comments, PR descriptions from other people, web pages, dependency READMEs, and client files.

## Security rules

### Triggers

1. Agents start only from: the dispatcher (cron), the `idea` label in `hq`, or `@claude` from **your** account.
2. Check the actor in every workflow: `if: github.actor == 'quythai'` (or a list of trusted maintainers). Never trust actors that end in `[bot]`.
3. Public repos: never trigger on `issues.opened` or `issue_comment` from any user.
4. Client repos that accept issues from client staff: use gh-aw (read-only agent + safe-outputs) for triage, or keep the project at L2.

### Versions

5. Pin `anthropics/claude-code-action` by **commit SHA**, at v1.0.94 or later.
6. Pin Claude Code at 2.1.128 or later.
7. Pin OpenSpec and promptfoo exact versions.
8. Renovate or Dependabot opens upgrade PRs. These PRs are always `risk:high`.

### Permissions

9. Workflow `permissions:` lists only what the job needs: `contents: write`, `pull-requests: write`, `issues: write`, `actions: read`. No `id-token` unless you use OIDC.
10. Cross-repo work (chief engineer opens PRs in project repos) uses a **GitHub App** with access to the listed repos only. Do not use a personal access token with all-repo scope.
11. Reviewer and verify jobs are read-only: `contents: read`, `pull-requests: write` (for comments only).

### Secrets

12. Agent jobs get only `ANTHROPIC_API_KEY` and the GitHub App token.
13. Production secrets live in GitHub **Environments** with required reviewers. Agent jobs never use these environments.
14. Hooks block reads of `.env*`, `*.pem`, `*.key`, `~/.ssh`, `/proc/*`.

### Tools

15. `--allowedTools` is an allow-list per project. Example for Spring Boot:
    `Read, Edit, Write, Glob, Grep, Bash(./mvnw:*), Bash(openspec:*), Bash(git:*), Bash(gh pr:*), Bash(gh issue comment:*)`
16. Hooks deny: `git push --force`, `git push origin main`, `rm -rf`, `curl`/`wget` to any host not on a list, changes to `.github/workflows/`.
17. **Agents cannot edit workflow files or the risk policy.** A PR that touches `.github/` or `hq/rules/` is always `risk:high` (CODEOWNERS + branch protection).

## Risk policy (`hq/rules/risk-policy.md`)

A change is `risk:high` if **any** item is true:

- It changes a public API contract, a DB schema, or a migration.
- It touches auth, security config, payments, PII, or secrets handling.
- It touches more than 1 service, or it changes infrastructure code.
- It changes CI workflows, the risk policy, hooks, or gates.
- It upgrades a dependency major version.
- The project has `auto_merge: never` (client work).
- The content will be published under your name to a new channel, or it makes claims about clients, money, or health.

Everything else is `risk:low`.
The chief engineer sets the level. The reviewer agent can raise it. Nothing can lower it except you.

## Cost controls

| Control | Where | Value (start) |
|---|---|---|
| Max turns per run | `claude_args: --max-turns` | 30 (code), 15 (content) |
| Job timeout | `timeout-minutes` | 45 |
| Agents per repo at the same time | Dispatcher | 1 |
| Agents across portfolio at the same time | Dispatcher | 3 |
| Fix attempts per PR | ci-fix workflow | 2 |
| Sprint budget per project | `portfolio.yml` | See file |
| Runs per day per project | Dispatcher | 10 |
| Model | `--model` | Default model for implementation. A smaller model for digest, labels, and simple content checks. |

How to measure cost:
- Use one **Anthropic Console workspace per project** (or per project group) with a spend limit. This gives a hard ceiling outside your YAML.
- The agent-run workflow writes the run's token usage to the board's Cost field.

Use an **API key** for unattended runs. A subscription OAuth token has rate limits that stop parallel unattended runs at the wrong time.

## Failure handling

| Failure | Detection | Response |
|---|---|---|
| Agent loops or hangs | Timeout, max turns | Job fails → `needs-human` |
| Anthropic API outage | Run error rate | Dispatcher pauses 1 hour, retries. Digest shows it. |
| claude-code-action regression | Many runs fail with the same error | Keep the pinned SHA. Run the skill locally until fixed. |
| Bad auto-merge | Bug report, failed deploy | Revert. Move that change type to `risk:high`. Add a regression test. |
| Spec drift (code changed, spec not) | CI archive check, `openspec validate` | Block merge |
| Runaway cost | Workspace limit, sprint budget | Dispatcher stops for the project |
| Prompt injection suspected | Agent output contains unexpected URLs, secret-like strings, or workflow edits | Hook blocks. Job fails. `p0` alert. |

## Kill switch

- Organization variable `AGENTS_ENABLED=false` (on a personal account, set the same repo variable in each repo with a small script) stops the dispatcher, the idea flow, and all scheduled agents in every repo.
- Every reusable workflow checks it first.
- Use it during incidents, holidays, or when Anthropic reports problems.
