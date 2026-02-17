---
name: lifecycle
description: Use when a project spans multiple phases with review gates between them — architecture then implementation then integration, for example
---

# Lifecycle Orchestration

Multi-phase project execution with review gates. Wraps `orchestrating-work` with phase sequencing, dependency waves, and architect-led planning.

**Core principle:** Plan phases with an architect. Execute each phase via `orchestrating-work`. Gate transitions with reviewer approval.

## When to Use

**Use lifecycle when:**

- Project has 2+ distinct phases (design → implement → integrate)
- Phases depend on each other (phase 2 needs phase 1's output)
- Quality gates matter (review before advancing)
- An architect should shape the plan

**Use `/orchestrate` instead when:**

- Single-pass decomposition suffices
- No phase dependencies
- No review gates needed
- Task completes in one wave

## The Pattern

### Step 1: Architect Plans Phases

Dispatch an architect worker to decompose the project into ordered phases:

```
Task tool (general-purpose):
  description: "Architect: Plan phases for [project]"
  prompt: [Use ./workers/architect-prompt.md from orchestrating-work]
```

The architect returns named phases with:

- Clear scope per phase
- Dependencies between phases
- Acceptance criteria for each phase gate

### Step 2: Create Phase Tasks

Encode phase identity in CC task subjects and use gate tasks as barriers:

```
TaskCreate: "[Phase 1: Foundation] Set up project structure"
TaskCreate: "[Phase 1: Foundation] Define data models"
TaskCreate: "[Phase 1: Foundation] Write schema migrations"

TaskCreate: "[Gate] Review Phase 1: Foundation"
  → addBlockedBy: [all phase-1 task IDs]

TaskCreate: "[Phase 2: API] Implement auth endpoints"
  → addBlockedBy: [gate-task-id]
TaskCreate: "[Phase 2: API] Implement CRUD endpoints"
  → addBlockedBy: [gate-task-id]

TaskCreate: "[Gate] Review Phase 2: API"
  → addBlockedBy: [all phase-2 task IDs]
```

Gate tasks block subsequent phases until the reviewer approves.

### Step 3: Execute Each Phase

For each phase, apply the `orchestrating-work` pattern:

1. **Build waves** — Group tasks whose dependencies are satisfied
2. **Dispatch workers** — Use coder/researcher/tester workers from `orchestrating-work`
3. **Synthesize results** — Verify all phase tasks complete

Parallel tasks within a wave run concurrently. Sequential dependencies enforce ordering.

### Step 4: Review Gate

When all tasks in a phase complete, the gate task activates. Dispatch a reviewer:

```
Task tool (general-purpose):
  description: "Reviewer: Gate review for Phase N"
  prompt: [Use reviewer-prompt.md with phase acceptance criteria]
```

**Verdicts** (match existing reviewer vocabulary):

| Verdict         | Meaning                         | Action                                     |
| --------------- | ------------------------------- | ------------------------------------------ |
| `APPROVE`       | Phase meets acceptance criteria | Mark gate complete; next phase unblocks    |
| `NEEDS_CHANGES` | Fixable issues found            | Create fix tasks in current phase; re-gate |
| `REJECT`        | Fundamental rework needed       | Revisit architect plan for this phase      |

On `APPROVE`, the gate task completes, unblocking all next-phase tasks via CC's `addBlockedBy` resolution.

On `NEEDS_CHANGES`, create new tasks for the fixes, add them as blockers to the gate, and re-run the phase.

### Step 5: Advance or Complete

After the final phase gate approves, run a completion audit:

1. Verify all phases passed their gates
2. Check project-level acceptance criteria
3. Report summary to user

## Dependency Waves

Within each phase, build execution waves from the task dependency graph:

1. Find all tasks with no unsatisfied dependencies → **Wave 1**
2. After Wave 1 completes, find newly unblocked tasks → **Wave 2**
3. Repeat until no pending tasks remain

See `reference/dependency-waves.md` for the algorithm.

## Phase Gates

Gate tasks are CC tasks with a special convention:

- Subject starts with `[Gate]`
- Blocked by all tasks in the current phase
- Blocks all tasks in the next phase
- Completion requires reviewer approval

See `reference/phase-gates.md` for the gate pattern.

## State

All state lives in CC's task list. No custom state files.

- Phase identity: encoded in task subject `[Phase N: Title]`
- Dependencies: CC's `addBlockedBy` / `addBlocks`
- Progress: CC task statuses (`pending` / `in_progress` / `completed`)
- Gates: `[Gate]` prefix tasks

To check progress, read the task list and group by phase prefix.

## Example

```
User: /mission Build a CLI tool with auth, config management, and plugin system

--- STEP 1: ARCHITECT ---

Architect plans:
  Phase 1: Foundation — project structure, config system, CLI framework
  Phase 2: Auth — login/logout, token storage, session management
  Phase 3: Plugins — plugin loader, registry, lifecycle hooks
  Phase 4: Integration — end-to-end flows, error handling, docs

--- STEP 2: CREATE TASKS ---

[Phase 1: Foundation] Scaffold project with TypeScript + Commander
[Phase 1: Foundation] Implement config file loading and validation
[Phase 1: Foundation] Set up test framework
[Gate] Review Phase 1: Foundation
  → blockedBy: all Phase 1 tasks

[Phase 2: Auth] Implement login flow with OAuth
  → blockedBy: gate-1
[Phase 2: Auth] Implement token storage
  → blockedBy: gate-1
[Gate] Review Phase 2: Auth
  → blockedBy: all Phase 2 tasks

... (phases 3, 4 follow same pattern)

--- STEP 3-4: EXECUTE + GATE ---

Phase 1 tasks execute via orchestrating-work (waves of coder/researcher workers)
Gate fires → reviewer approves → Phase 2 unblocks
Repeat for each phase

--- STEP 5: COMPLETE ---

All gates passed. Completion audit confirms project criteria met.
```

## Integration

**Wraps:**

- `bach:orchestrating-work` — Each phase executes using the two-tier pattern

**Workers used:**

- `architect` — Plans phases (via `workers/architect-prompt.md`)
- `coder`, `researcher`, `reviewer`, `tester` — Execute phase tasks

**Invoked by:**

- `/mission` command — Entry point for lifecycle projects
