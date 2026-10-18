---
name: KB-context-handoff-contract
id: KB-context-handoff-contract
date: 2026-10-13
status: active
parent: ../README.md
---

# KB: Context Handoff Contract

## Purpose

Each Claude Code session is stateless. Without a structured handoff payload,
every session starts blind and makes assumptions. This contract defines the
MINIMUM required context that must be passed to every session spawn.

A session that does not receive a valid handoff payload MUST refuse to proceed
and emit: `HANDOFF_ERROR: missing required fields: [list]`

The payload points to the OpenSpec change. It never copies the spec.

## Payload Schema

```json
{
  "session_type": "intake | planning | implementing | review-resolving",
  "project_id": "modern-java",
  "connector": "github",
  "repo": "thaiquangquy/ModernJava",
  "local_path": "~/workspace/quy/ModernJava",
  "base_branch": "develop",
  "type": "content",
  "change_id": "add-records-chapter",
  "change_path": "openspec/changes/add-records-chapter/",
  "issue": 42,
  "pr": null,
  "risk": "low",
  "priority": "p2",
  "depends_on": [],
  "gates": ["mdbook-build", "summary-lists-new-pages", "markdownlint"],
  "openspec_version": "<exact pinned version>",
  "relevant_files": [
    "src/records.md",
    "src/SUMMARY.md"
  ],
  "commit_batch_rule": "1 commit per scenario in the change's spec deltas",
  "session_context": {
    "previous_session_summary": "Optional: what the last session did",
    "open_questions": [],
    "blocked_on": null
  }
}
```

Fields `project_id`, `connector`, `repo`, `base_branch`, `type`, and `gates` come from `portfolio.yml`.
Fields `risk`, `priority`, and `depends_on` come from `openspec/changes/<change-id>/agent.yml`.

## Session-Type Specific Requirements

### `intake` session (P-01)
Required: `issue`, `repo`
Output: a validated brief passed to P-02, or `needs-human` + comment

### `planning` session (P-02)
Required: `project_id`, `repo`, and either the chosen option from C-00 or the P-01 brief
Output: `openspec/changes/<change-id>/` + `agent.yml` on branch `spec/<change-id>`, spec PR opened

### `implementing` session (P-03 → P-04 → P-05)
Required: `change_id`, `change_path`, `issue`, `gates`, `relevant_files[]`, `commit_batch_rule`
Pre-condition: the change folder exists on `base_branch` and `openspec validate <change-id> --strict` passes
Output: implementation PR from `agent/<change-id>` with the change archived

### `review-resolving` session (P-06)
Required: `change_id`, `pr`, `review_comments[]`
Output: new commit pushed to the PR branch
Constraint: MUST NOT change the spec deltas or break existing passing checks

## review_comments Schema (for review-resolving)

```json
"review_comments": [
  {
    "author": "thaiquangquy",
    "file": "src/records.md",
    "line": 42,
    "comment": "Use a shorter example here",
    "type": "suggestion | question | blocker | nitpick"
  }
]
```

Only `suggestion` and `blocker` types require changes.
`question` types require a reply only.

## Where the payload lives

| Runner | Location |
|---|---|
| GitHub (claude-code-action) | Built by the dispatcher from `portfolio.yml` + `agent.yml` + the issue, and passed in the prompt. Not committed. |
| Local session | `runs/<project_id>/<change-id>/handoff-<session_type>-<timestamp>.json` in this repo (git-ignored), so a session can be re-run with identical context |

## Changelog
- 2026-10-13 — v2.0 — `plan_md_path` → `change_id`/`change_path`; add `project_id`, `connector`, `risk`, `gates`; payload no longer lives in `spec/`
- 2026-06-09 — v1.0 — initial version
