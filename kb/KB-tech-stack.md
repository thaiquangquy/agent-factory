---
name: KB-tech-stack
id: KB-tech-stack
date: 2026-06-09
status: active
parent: ../README-kb.md
---

# KB: Tech Stack Per Repo

## Code project (template)

Generic values for a Java/Gradle + Flyway + jOOQ backend with a React UI. Copy per repo when such a project joins.

### backend (`repo:api`)

| Field           | Value                                      |
|-----------------|--------------------------------------------|
| Language        | Java / Kotlin                              |
| Build tool      | Gradle                                     |
| Test framework  | JUnit 5 + Mockito                          |
| Unit test cmd   | `./gradlew test`                           |
| Integration cmd | `./gradlew integrationTest`                |
| ORM             | jOOQ (generated classes in `src/generated`)|
| Codegen cmd     | `./gradlew generateJooq`                   |

### frontend (`repo:web`)

| Field           | Value                                      |
|-----------------|--------------------------------------------|
| Language        | TypeScript                                 |
| Framework       | React                                      |
| Unit test cmd   | `npm test`                                 |
| E2E test cmd    | `npx playwright test`                      |

### database (`repo:db`)

| Field              | Value                                   |
|--------------------|-----------------------------------------|
| Migration tool     | Flyway                                  |
| Migration location | `src/main/resources/db/migration/`      |
| Naming convention  | `V{version}__{description}.sql`         |
| Validate cmd       | `./gradlew flywayValidate`              |
| Migrate cmd        | `./gradlew flywayMigrate`               |

## jOOQ Codegen Sequence (CRITICAL)

When `repo:db` label is present, the bot MUST follow this exact sequence:

```
1. Write migration file → repos/db/src/main/resources/db/migration/V{next}__{description}.sql
2. Run: cd repos/db && ./gradlew flywayMigrate
3. Run: cd repos/api && ./gradlew generateJooq
4. Verify: src/generated/jooq/ has updated classes
5. ONLY THEN begin backend implementation
```

Skipping any step will cause compile errors in backend. This is a hard dependency.

## modern-java

Pilot project (`portfolio.yml` id `modern-java`). mdBook, markdown only: no Java build and no unit tests.
A spec scenario is verified by a structure/build check, not by JUnit.

| Field              | Value                                                        |
|--------------------|--------------------------------------------------------------|
| Type               | content (mdBook)                                             |
| Content location   | `src/` (`src/<topic>.md` + `src/<topic>/*.md`)               |
| Table of contents  | `src/SUMMARY.md` (a page not listed here is not built)       |
| Build cmd          | `mdbook build`                                               |
| TOC check          | Every new `src/**/*.md` in the diff appears in `src/SUMMARY.md` outside `<!-- -->` |
| Lint cmd           | `npx markdownlint-cli --config .markdownlint.yaml <changed files>` |
| Typo check         | LanguageTool via `reviewdog/action-languagetool` in CI (advisory, `level: info`) |
| Local preview      | `node dev/serve.mjs` (needs JDK 25; dev only)                |
| Base branch        | `develop` (deploy branch `master` is owner only)             |

Blocking gates: build, TOC check, lint. LanguageTool is advisory.

## Per-Project Override

Project-specific values live in `portfolio.yml` (`gates`, `base_branch`, `max_turns`) and in the project's own `CLAUDE.md`.
If they disagree with this KB, `portfolio.yml` wins for the factory, and the project's `CLAUDE.md` wins for commands.

## Changelog
- 2026-10-18 — v1.2 — per-repo sections become one generic code-project template
- 2026-10-13 — v1.1 — add modern-java; overrides move to portfolio.yml
- 2026-06-09 — v1.0 — initial version
