---
description: Generate specs for an entire project incrementally, module by module.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

Systematically reverse-engineer specs for an entire codebase. Generates an index spec with architecture overview, then iterates through modules producing individual .spec.md files.

## Execution Flow

### 1. Project Survey

Scan the project root to understand structure:

1. **Detect project type**: monorepo, single app, library, CLI, etc.
2. **Identify top-level modules**: `src/` subdirectories, packages in monorepo, major feature folders
3. **Estimate total scope**: file count, LOC per module
4. **Detect existing specs**: Check `specs/` directory for already-generated specs

### 2. Generate Batch Plan

Present the analysis plan to the user:

```markdown
## Reverse-Spec Batch Plan

**Project**: <name>
**Type**: <project type>
**Total scope**: N files, ~N LOC

### Modules to analyze (in dependency order):

| # | Module | Files | LOC | Existing Spec? | Priority |
|---|--------|-------|-----|----------------|----------|
| 1 | core/utils | 5 | 320 | No | Foundation |
| 2 | models/ | 8 | 890 | No | Data layer |
| 3 | auth/ | 12 | 1450 | Yes (outdated) | Critical |
| ... | | | | | |

**Estimated effort**: ~N minutes with AI analysis

Proceed with all? Or select specific modules (e.g., "1,3,5" or "auth/ models/")
```

Wait for user confirmation before proceeding.

### 3. Generate Index Spec

Create `specs/_index.spec.md`:

```markdown
---
type: project-index
version: 1.0
generated_by: reverse-spec-batch
last_updated: <date>
---

# <Project Name> — Architecture Overview

## System Purpose
<High-level what this project does>

## Architecture
<Pattern: monolith/microservice/serverless/etc>
<Key architectural decisions visible in code>

## Module Map
| Module | Spec | Purpose | Dependencies |
|--------|------|---------|--------------|
| <mod>  | [link](./mod.spec.md) | <purpose> | <deps> |

## Cross-Cutting Concerns
- Authentication: <how handled>
- Error handling: <pattern>
- Logging: <approach>
- Configuration: <approach>

## Technology Stack
- Language(s): ...
- Framework(s): ...
- Database(s): ...
- Key libraries: ...
```

### 4. Iterate Through Modules

For each module in the plan:

1. Run the equivalent of `/reverse-spec <module-path>`
2. Write to `specs/<module-name>.spec.md`
3. Update `specs/_index.spec.md` module map with link
4. Report: "✅ Generated spec for <module> (N/M complete). Continue?"
5. If user says stop, save progress and report remaining modules

### 5. Final Summary

After all modules (or user stop):

```markdown
## Batch Reverse-Spec Complete

**Generated**: N/M specs
**Index**: specs/_index.spec.md
**Total time**: ~N minutes

### Generated Specs:
- ✅ specs/auth.spec.md (high confidence)
- ✅ specs/models.spec.md (medium confidence)
- ⏭️ specs/api.spec.md (skipped by user)

### Project-Wide Observations:
- <Cross-module patterns noticed>
- <Shared technical debt themes>
- <Architecture recommendations>
```

## Guidelines

- Process modules in **dependency order** (foundations first)
- **Resume-friendly**: If interrupted, check existing specs and skip completed modules
- **Don't re-generate** specs that already exist unless user requests with `--force`
- Keep each module spec **self-contained** but cross-referenced via the index
