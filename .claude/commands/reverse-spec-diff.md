---
description: Compare existing code against its .spec.md to find drift — new behaviors not in spec, or spec items no longer in code.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

Detect specification drift between a .spec.md and its corresponding source code. Useful after code changes to keep specs current, or before refactoring to understand what's changed.

## Execution Flow

### 1. Parse Arguments

Expected format: `<spec-file> [source-target]`

- `<spec-file>`: Path to existing .spec.md (required)
- `[source-target]`: Path to source code (optional — inferred from spec's `related_files` frontmatter)

If spec file doesn't exist, suggest running `/reverse-spec` first.

### 2. Load & Parse Spec

Read the .spec.md and extract:
- All documented interfaces (function signatures, endpoints, types)
- All documented business rules
- All documented edge cases
- All documented constraints
- Technical debt items
- File inventory from appendix

### 3. Analyze Current Code

Re-scan the source target using the same analysis as `/reverse-spec` but keep results in memory (don't write).

### 4. Compute Diff

Compare spec vs current code:

#### Added in Code (not in spec)
- New public APIs / exports
- New business logic branches
- New error handling
- New dependencies
- New files not in inventory

#### Removed from Code (still in spec)
- APIs documented but no longer exported
- Business rules documented but logic removed
- Files in inventory but deleted

#### Modified
- Signature changes (parameters added/removed/renamed)
- Logic changes (different branching, new conditions)
- Constraint changes (different limits, new validations)

### 5. Output Drift Report

```markdown
# Spec Drift Report: <component name>

**Spec**: <spec-file path>
**Source**: <source path>
**Analyzed**: <current date>

## Summary
- Additions (code → spec needed): N
- Removals (spec → code deleted): N  
- Modifications: N
- Drift severity: LOW | MEDIUM | HIGH

## Additions (in code, missing from spec)
| Item | Type | Location | Description |
|------|------|----------|-------------|

## Removals (in spec, missing from code)
| Item | Spec Section | Description |
|------|-------------|-------------|

## Modifications (changed since spec)
| Item | Spec Says | Code Shows | Location |
|------|-----------|------------|----------|

## Recommendation
<Whether to update spec, update code, or both>
```

### 6. Offer Update

Ask: "Would you like me to update the spec to match current code? This will run `/reverse-spec` with the existing spec as a baseline."

Do NOT auto-update. Wait for user confirmation.

## Guidelines

- This is a **read-only analysis** — no file modifications without user consent
- Focus on **semantic** differences, not formatting/style changes
- Ignore whitespace, comment-only changes, and import reordering
- Flag **breaking changes** (removed/modified public APIs) prominently
