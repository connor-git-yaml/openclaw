---
description: Reverse-engineer a structured .spec.md from existing source code. Supports single files, directories, or entire modules.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

Analyze existing source code and generate a structured specification document (.spec.md) that captures intent, interfaces, business logic, constraints, edge cases, and technical debt. The generated spec is compatible with spec-kit's spec-driven development workflow, enabling future forward-engineering iterations.

## Execution Flow

### 1. Parse Target

Interpret `$ARGUMENTS` to determine the analysis target:

- **Single file**: e.g., `src/auth/login.ts`
- **Directory**: e.g., `src/auth/` — analyze all source files recursively
- **Module/pattern**: e.g., `src/auth/**/*.ts` or a logical module name
- **No argument**: Analyze the entire project (warn user about scope, ask for confirmation)

If the target doesn't exist, ERROR with suggestions based on project structure.

### 2. Determine Output Location

- Default: `specs/<target-name>.spec.md` (relative to project root)
- If `--out <path>` is specified in arguments, use that path
- If target is a directory, output to `specs/<dirname>.spec.md`
- If target is a file, output to `specs/<filename-without-ext>.spec.md`
- Create the `specs/` directory if it doesn't exist

### 3. Scan & Inventory

For the target scope, build an inventory:

1. **List all source files** in scope (skip binary, node_modules, vendor, build artifacts)
2. **Detect language(s)** and framework(s) in use
3. **Identify entry points**: exports, main functions, route handlers, class constructors
4. **Map dependencies**: imports, injections, inherited classes, external packages
5. **Estimate complexity**: file count, total LOC, cyclomatic complexity (rough estimate)

If scope exceeds ~50 files or ~5000 LOC, switch to **incremental mode** (see Section 8).

### 4. Extract Architectural Overview

Analyze code structure to determine:

- **Component type**: library, service, CLI tool, UI component, middleware, data model, etc.
- **Architectural pattern**: MVC, event-driven, pipeline, repository pattern, etc.
- **Key abstractions**: primary classes/interfaces/types and their relationships
- **Data flow**: how data enters, transforms, and exits the component

### 5. Deep Analysis — Extract Spec Sections

For each major unit (class, module, significant function), extract:

#### 5a. Intent (意图)
- What problem does this code solve?
- Infer from: function/class names, comments, docstrings, README references, test descriptions
- If unclear, state the inferred intent and mark with `[INFERRED]`

#### 5b. Interface (接口定义)
- Public API: exported functions, class methods, REST/GraphQL endpoints, CLI commands
- Input types and parameters (with defaults if present)
- Return types and output shapes
- Events emitted or consumed
- Configuration options / environment variables read

#### 5c. Business Logic (业务逻辑)
- Core algorithms and decision trees
- State machines or workflow steps
- Validation rules (input validation, business rule validation)
- Transformation pipelines
- Conditional branches with business significance

#### 5d. Data Structures (数据结构)
- Type definitions, interfaces, schemas
- Database models/migrations if present
- API request/response shapes
- Internal state shapes

#### 5e. Constraints (约束)
- Performance characteristics (timeouts, rate limits, batch sizes)
- Security measures (auth checks, sanitization, encryption)
- Resource limits (memory, connections, file size)
- Platform/environment requirements
- Invariants maintained by the code

#### 5f. Edge Cases (边界条件)
- Error handling patterns (try/catch, Result types, error codes)
- Null/undefined/empty handling
- Boundary conditions in loops, pagination, recursion
- Race conditions or concurrency handling
- Graceful degradation / fallback behavior
- Identified from: catch blocks, guard clauses, default cases, test edge cases

#### 5g. Technical Debt (技术债务)
- TODO/FIXME/HACK/XXX comments
- Suppressed linting rules (eslint-disable, @ts-ignore, noinspection)
- Dead code (unreachable branches, unused exports)
- Copy-pasted logic (near-duplicate blocks)
- Missing error handling (bare catches, swallowed errors)
- Outdated dependencies or deprecated API usage
- Missing tests for critical paths
- Hardcoded values that should be configurable
- Overly complex functions (high cyclomatic complexity)

### 6. Cross-Reference with Tests

If test files exist for the target:

1. Map test cases to spec sections (which behaviors are tested?)
2. Identify **untested paths** — business logic without corresponding tests
3. Extract **implicit requirements** from test assertions that aren't obvious in source
4. Note test quality: are tests unit/integration/e2e? Mock-heavy? Brittle?

### 7. Generate .spec.md

Write the spec file using this template:

```markdown
---
type: component-spec
version: 1.0
generated_by: reverse-spec
source_target: <target path>
related_files:
  - <list of analyzed files>
last_updated: <current date YYYY-MM-DD>
confidence: <high|medium|low — based on code clarity and documentation>
---

# <Component Name> Spec

> Auto-generated reverse specification from existing code.
> Review and refine before using for forward development.

## 1. Intent (意图)

<What this component does and why it exists>

## 2. Interface (接口定义)

### Public API

<Exported functions, methods, endpoints with signatures>

### Configuration

<Environment variables, config options>

### Events / Signals

<Events emitted or consumed, if any>

## 3. Business Logic (业务逻辑)

<Core algorithms, decision trees, workflows>

### Key Rules

<Numbered list of business rules extracted from code>

## 4. Data Structures (数据结构)

<Type definitions, schemas, models>

## 5. Constraints (约束)

### Performance
<Timeouts, limits, batch sizes>

### Security
<Auth, sanitization, encryption>

### Platform
<Environment requirements, dependencies>

## 6. Edge Cases (边界条件)

<Error handling patterns, boundary conditions, fallbacks>

| Condition | Handling | Location |
|-----------|----------|----------|
| <edge case> | <how it's handled> | <file:line> |

## 7. Technical Debt (技术债务)

| Item | Severity | Location | Description |
|------|----------|----------|-------------|
| <debt item> | HIGH/MED/LOW | <file:line> | <description> |

## 8. Test Coverage (测试覆盖)

- **Tested**: <list of tested behaviors>
- **Untested**: <list of identified gaps>
- **Test Quality Notes**: <observations>

## 9. Dependencies (依赖关系)

### Internal
<Other project modules this depends on>

### External
<Third-party packages with versions>

## Appendix: File Inventory

| File | LOC | Primary Purpose |
|------|-----|-----------------|
| <file> | <loc> | <purpose> |
```

### 8. Incremental Mode (Large Codebases)

When scope exceeds thresholds (~50 files or ~5000 LOC):

1. **Generate index spec first**: `specs/_index.spec.md` with high-level architecture overview
2. **Break into sub-specs**: One .spec.md per major directory or module
3. **Report progress**: After each sub-spec, report what's done and what remains
4. **Cross-reference**: Each sub-spec links to related sub-specs
5. **Ask user**: "Generated spec for `src/auth/`. Continue with `src/api/`?" (proceed unless stopped)

### 9. Quality Self-Check

Before finalizing, validate:

- [ ] All public interfaces documented
- [ ] No `[INFERRED]` markers without justification
- [ ] Technical debt items have severity ratings
- [ ] Edge cases table is populated (not empty)
- [ ] File inventory matches actual analyzed files
- [ ] Frontmatter `related_files` is accurate

Report any items that couldn't be fully analyzed with reasons.

### 10. Completion Report

Output a summary:

```
✅ Reverse spec generated: specs/<name>.spec.md

📊 Analysis Summary:
- Files analyzed: N
- Total LOC: N
- Public APIs found: N
- Business rules extracted: N
- Edge cases identified: N
- Technical debt items: N
- Test coverage gaps: N
- Confidence: high|medium|low

💡 Next steps:
- Review and refine the generated spec
- Use /speckit.plan to create implementation plan from spec
- Use /speckit.tasks to break down into tasks
```

## Guidelines

- **Be honest about uncertainty**: Use `[INFERRED]` when guessing intent, `[UNCLEAR]` when code is ambiguous
- **Preserve developer context**: Include relevant code comments in the spec
- **Don't over-abstract**: Keep specs concrete and traceable to actual code
- **Language-agnostic output**: The spec format works regardless of source language
- **Respect .gitignore**: Don't analyze ignored files unless explicitly targeted
- **No modification**: This command is READ-ONLY — it never modifies source code
