---
name: cartograph
description: >-
  Produces an accurate repository map (structure, modules, entry points,
  relationships) and writes it to docs/CODEBASE_MAP.md. Use when the user asks
  for a codebase map, project map, architecture overview, onboarding doc for
  the repo layout, or mentions Cartograph / CODEBASE_MAP.
---

# Cartograph — codebase map

## Goal

Produce a single markdown document that helps humans and agents navigate the repository. **Save the result only to `docs/CODEBASE_MAP.md`** (create `docs/` if it does not exist). Replace or refresh that file completely unless the user asks for an incremental update.

## Rules

1. **Ground truth only**: Infer structure from the filesystem and from files you read (`settings.gradle`, `build.gradle`, package layout, README, CI config). Do not invent modules, paths, or dependencies.
2. **Scope to this repo**: Describe what exists here; note external services only when the code clearly integrates them (config keys, clients, README).
3. **Stay current**: Prefer scanning the tree and opening key files over relying on chat memory from older sessions.
4. **Concise map, not a tutorial**: Orient readers; deep dives belong in separate docs or wiki links if they already exist.

## Discovery workflow

Work through these in order; skip steps that do not apply (e.g. non-Gradle projects).

1. **Root & tooling**
   - Identify build system(s), language(s), and top-level config (e.g. Gradle/Maven, `package.json`, Docker, CI under `.github/`).
   - List notable root files (README, CONTRIBUTING, license, compose files).

2. **Repository structure**
   - Summarize top-level directories and their roles (application code, libraries, infra, docs, scripts).
   - For multi-module projects, read `settings.gradle` / workspace definition and list each included module.

3. **Major modules / packages**
   - For each module (or bounded area), give a short purpose statement.
   - Note important package or directory groupings (e.g. `features/`, `config/`, `api/`).

4. **Key entry points**
   - Find application bootstrap (`main`, framework entry), servers, CLI commands, scheduled jobs, and Discord/bot listeners if applicable — cite paths.
   - Mention primary configuration files (templates, env expectations) without dumping secrets.

5. **Important relationships**
   - Module dependency direction (who depends on whom).
   - Runtime flows worth naming (e.g. command dispatch → handler → service → persistence).
   - Shared libraries vs. deployable artifact(s).

6. **Cross-check**
   - Reconcile module list with actual directories.
   - If something is ambiguous, say so briefly rather than guessing.

## Output: `docs/CODEBASE_MAP.md`

Use this skeleton; adapt headings only when the project shape demands it (e.g. monorepo with multiple apps).

```markdown
# Codebase map

Brief one-paragraph overview of what this repository is for.

## Repository structure

- Bullet list of top-level directories/files with one-line descriptions.

## Modules / major areas

### [module-or-area-name]

- **Purpose**: …
- **Location**: …
- **Notes**: …

(Repeat per module or coherent slice of the tree.)

## Key entry points

| Kind | Path | Role |
|------|------|------|
| … | `…` | … |

## Configuration & environment

- Where config lives; how to run locally or tests if documented in-repo.

## Relationships & data flow

- Dependency graph or bullet hierarchy (module A → module B).
- Short narrative of main request/event/command flows.

## Related documentation

- Links/paths to README, wiki, CONTRIBUTING, or design docs in this repo.

## Map metadata

- Generated for commit / date (if known from git or user context): …
```

## Quality bar

- Paths use forward slashes and are relative to the repo root.
- Prefer tables or bullets over long prose.
- No duplicated README content; **summarize** and point to the source file.
