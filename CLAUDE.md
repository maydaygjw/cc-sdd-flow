# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**cc-sdd-flow** is a Specification Driven Development (SDD) workflow framework and methodology documentation for AI-assisted software development. This is a meta-framework that provides templates, commands, and standards to guide AI agents through structured, specification-first software development.

## Repository Structure

```
/
├── commands/              # Command definitions for AI agents
│   ├── generate_plan.md   # Generate technical implementation plans
│   └── generate_task.md   # Generate atomic task lists
├── meta/                  # Core methodology and standards
│   ├── constitution.md    # Development principles (HIGHEST AUTHORITY)
│   ├── CLAUDE.md          # SDD workflow methodology
│   └── AGENT_SAMPLE.md    # Technical standards for backend/frontend
└── templates/             # Document templates
    ├── spec-template.md   # Business specification template
    ├── plan-template.md   # Technical plan template
    └── task-template.md   # Task decomposition template
```

## Core Concepts

### The SDD Workflow (3 Phases)

1. **Phase 1 - Specification** (`spec.md`)
   - Business requirements in plain language
   - Focuses on "What" and "Why", not "How"
   - 2-3 pages, business-oriented
   - No technical implementation details

2. **Phase 2 - Planning** (`plan.md`)
   - Technical architecture and design
   - API design, database schema, tech stack
   - Generated using `commands/generate_plan.md`
   - Must follow `templates/plan-template.md` structure

3. **Phase 3 - Implementation** (`tasks.md`)
   - Atomic, executable task list
   - Generated using `commands/generate_task.md`
   - Must follow `templates/task-template.md` structure
   - Each task modifies exactly one file

### The Constitution (Highest Authority)

`meta/constitution.md` defines immutable development principles that supersede all other instructions:

1. **Simplicity First**: YAGNI principle, no over-engineering, minimal code
2. **Test-First Imperative**: Strict TDD (Red → Green → Refactor), non-negotiable
3. **Clarity and Explicitness**: Explicit error handling, no hidden state
4. **Single Responsibility**: Clear module boundaries, low coupling

**Critical**: Any plan or task generation must pass "constitutionality review" against these principles.

## Command Usage

### Generate Technical Plan

```bash
# Role: Chief Architect
# Input: spec.md, constitution.md, plan-template.md
# Output: plan.md
```

**Constraints**:
- Must follow `templates/plan-template.md` structure exactly
- Only design what's in spec.md (no feature additions)
- Must comply with constitution.md principles
- Include: architecture, tech stack, API design, database design, security, performance, deployment

### Generate Task List

```bash
# Role: Senior Technical Lead
# Input: spec.md, plan.md, constitution.md, task-template.md
# Output: tasks.md
```

**Constraints**:
- Each task modifies exactly ONE file
- Tasks must declare dependencies: `depends: [task_ids]`
- Parallel tasks marked with `[P]`
- TDD-first: test task before implementation task
- Follow structure: Infrastructure → Use Cases → Integration → E2E Tests

## Task Decomposition Principles

### Atomic Task Rules
- One task = one file modification or creation
- No compound tasks
- No vague tasks like "implement all features"

### Task Organization
1. **Use-case driven**: Start from business use cases
2. **Top-down**: Interface (router) → Service → DAO → Entity
3. **STUB strategy**: Use stubs for unimplemented dependencies
4. **TDD mandatory**: Write failing test first, then implementation

### Dependency Management
- Explicit dependencies: `depends: [1, 2, 3]`
- Parallel execution: Mark with `[P]` if no dependencies
- Tasks execute in dependency order

## Architecture Standards (from AGENT_SAMPLE.md)

### Layered Architecture
```
interface/routes/    → RESTful API endpoints (returns JSON)
services/           → Business logic (returns Schema objects)
dao/                → Data access (returns Entity objects)
entity/             → SQLAlchemy ORM models
schema/             → Pydantic validation models
api/                → External API wrappers (returns Schema objects)
utils/              → Shared utilities
```

### Data Flow Patterns
- **Database flow**: Route → Service → DAO → Entity → Service (convert to Schema) → Route (.model_dump()) → JSON
- **External API flow**: Route → Service → API → External → Schema → Service → Route (.model_dump()) → JSON

### Type Requirements
- DAO layer: Returns `Entity` objects (never Dict)
- Service layer: Returns `Schema` objects (never Dict)
- API layer: Returns `Schema` objects (never Dict)
- Interface layer: Calls `.model_dump()` to return JSON

## Testing Standards

### Test-First Imperative (Non-Negotiable)
- **TDD Cycle**: Red → Green → Refactor
- Write failing test first, then minimal code to pass
- Refactor under test protection

### Test Categories
1. **Unit Tests**: Verify single functions/classes
   - Cover: happy path, branches, exceptions
   - **No mocks allowed** - use real dependencies or lightweight fakes

2. **Integration Tests**: Verify module collaboration
   - Cover: main flows, external system interactions
   - Mock external dependencies only (not internal logic)

3. **End-to-End Tests**: Verify complete user scenarios
   - Cover: critical business flows
   - **No mocks allowed** - use real environment or containers

### Quality Requirements
- Test coverage >= 80%
- Tests must verify real business scenarios
- Use parameterized/data-driven tests to avoid duplication

## Technology Stack (Sample Standards)

### Backend
- Python 3.12+
- FastAPI (web framework)
- SQLAlchemy 2.x (ORM)
- Pydantic (validation)
- MySQL (database)
- Redis (cache/queue)
- pytest (testing)

### Frontend
- Vue.js 3
- Tailwind CSS

### Performance Targets
- API response time < 200ms
- MQTT processing < 100ms
- Concurrent connections > 1000 devices
- System availability > 99.5%

## Key Principles for AI Agents

1. **Never add features not in spec.md** - No "giving yourself extra work"
2. **Constitution.md is supreme** - It overrides all other instructions
3. **Test before code** - Every implementation must have a test first
4. **One task, one file** - Atomic task decomposition is mandatory
5. **Follow templates exactly** - Use provided templates for spec/plan/tasks
6. **Explicit dependencies** - Mark all task dependencies clearly
7. **Strong typing** - Use Entity/Schema objects, not dicts
8. **Simple over clever** - Avoid over-engineering and abstractions

## Error Handling Philosophy

- **Default: Let exceptions propagate** - Don't catch unless necessary
- Only catch exceptions for:
  - Resource cleanup
  - Type conversion for interface contracts
  - Explicit error recovery
- Preserve stack traces and error context
- No silent failures or ignored errors

## Documentation Standards

- All docstrings in Chinese
- Code comments in Chinese (when needed)
- Comments explain "why", not "what"
- API/module documentation must be clear and accurate

## Governance

The `constitution.md` has **highest priority** and supersedes:
- CLAUDE.md
- AGENT_SAMPLE.md
- Single session instructions
- Any other documentation

All plans and tasks must undergo "constitutionality review" before generation.
