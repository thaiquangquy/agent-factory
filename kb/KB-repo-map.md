---
name: KB-repo-map
id: KB-repo-map
date: 2026-06-09
status: active
parent: ../README-kb.md
---

# KB: Repository Map

## Purpose
Projects and their connector, autonomy, and budget now live in [`portfolio.yml`](../portfolio.yml).
This file keeps the per-repo detail for multi-repo projects.

> **Status:** template. No multi-repo project is in `portfolio.yml` yet.
> Single-repo GitHub projects (like `modern-java`) need no row here.

## Label → Repo Mapping (template)

Fill in one row per repo when a multi-repo project joins the portfolio.

| Label      | Repo Path                       | Type     | Primary Language    | Test Command         |
|------------|---------------------------------|----------|---------------------|----------------------|
| `repo:db`  | `~/workspace/<project>/db`      | database | SQL/Flyway          | see KB-tech-stack.md |
| `repo:api` | `~/workspace/<project>/api`     | backend  | Java/Spring Boot    | see KB-tech-stack.md |
| `repo:web` | `~/workspace/<project>/web`     | frontend | TypeScript/React    | see KB-tech-stack.md |

## Multi-Repo Touch Rules

When a ticket touches multiple repos, the order of operations is FIXED:

```
db → (jOOQ codegen) → backend → frontend/mobile
```

Never implement backend before db migration + codegen is complete.
Never implement frontend before backend API contract is finalized.

With OpenSpec, one change = one repo. A cross-repo idea becomes one change per repo,
and the touch order is written as `depends_on` in each change's `agent.yml`:

```yaml
# openspec/changes/add-quota-api/agent.yml   (repo:api)
depends_on: [add-quota-table]               # the repo:db change must merge first
```

The dispatcher skips an issue while any change in its `depends_on` is still open.

## Label Conventions

- A ticket for a multi-repo project MUST have at least one `repo:*` label.
- No `repo:*` label → P-01 asks for clarification and halts.
- Several `repo:*` labels → C-00/P-02 write one change per repo, linked by `depends_on`.

## Repo Resolution Algorithm (for ticket-intaker)

1. Read all labels on the issue
2. Filter labels matching pattern `repo:*`
3. Look up each in the table above
4. Sort by touch order (db → backend → frontend/mobile)
5. Pass the resolved list to P-02, which writes one change per repo with `depends_on`

## Changelog
- 2026-10-18 — v1.2 — mapping table becomes a generic template
- 2026-10-13 — v1.1 — projects move to portfolio.yml; multi-repo project parked; touch order as depends_on
- 2026-06-09 — v1.0 — initial version
