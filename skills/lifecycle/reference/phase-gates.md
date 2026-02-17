# Phase Gates

A gate is a CC task that acts as a barrier between phases. It blocks until a reviewer approves the current phase's output.

## Convention

Gate tasks follow this naming convention:

```
[Gate] Review Phase N: <Phase Title>
```

## Structure

```
Phase 1 tasks ──┐
                 ├──▶ [Gate] Review Phase 1 ──▶ Phase 2 tasks
                 │         (blockedBy: all         (blockedBy: gate)
                 │          phase-1 tasks)
Phase 1 tasks ──┘
```

## Creating a Gate

```
# Create phase tasks first
TaskCreate: "[Phase 1: Foundation] Set up project"     → id: 1
TaskCreate: "[Phase 1: Foundation] Define models"       → id: 2
TaskCreate: "[Phase 1: Foundation] Write migrations"    → id: 3

# Create gate, blocked by all phase tasks
TaskCreate: "[Gate] Review Phase 1: Foundation"         → id: 4
TaskUpdate: id=4, addBlockedBy: [1, 2, 3]

# Create next phase, blocked by gate
TaskCreate: "[Phase 2: API] Implement endpoints"        → id: 5
TaskUpdate: id=5, addBlockedBy: [4]
```

## Triggering the Gate

When all phase tasks complete, the gate task unblocks automatically (CC resolves `addBlockedBy` when dependencies complete). The manager then:

1. Marks the gate task `in_progress`
2. Dispatches a reviewer with the phase's acceptance criteria
3. Processes the verdict

## Verdicts

Gate verdicts map directly to Bach's reviewer vocabulary:

| Verdict         | Reviewer status | Gate action                                       |
| --------------- | --------------- | ------------------------------------------------- |
| `APPROVE`       | `complete`      | Mark gate `completed`; next phase unblocks        |
| `NEEDS_CHANGES` | `partial`       | Create fix tasks; add as new blockers to gate     |
| `REJECT`        | `incapable`     | Re-plan phase with architect; rebuild phase tasks |

### APPROVE

The gate task marks `completed`. CC automatically unblocks all tasks that depend on it (the next phase).

### NEEDS_CHANGES

The reviewer identified fixable issues. The manager:

1. Creates new fix tasks: `[Phase N: Title] Fix: <issue>`
2. Adds fix tasks as blockers to the gate: `TaskUpdate: gate-id, addBlockedBy: [fix-task-ids]`
3. Dispatches fix tasks via `orchestrating-work`
4. When fixes complete, re-runs the gate review

### REJECT

Fundamental problems require rework. The manager:

1. Dispatches an architect to re-plan the phase
2. Creates replacement tasks for the rejected work
3. Links them to the gate as new blockers

## Review Prompt

When triggering a gate review, provide the reviewer with:

```
You are reviewing Phase N: <Title> for gate approval.

## Acceptance Criteria
<criteria from architect's phase plan>

## Phase Tasks Completed
<list of completed tasks with summaries>

## Artifacts
<files created/modified during this phase>

## Your Verdict
Evaluate whether the phase meets its acceptance criteria.
Return APPROVE, NEEDS_CHANGES, or REJECT per your standard output format.
```

## Multiple Gates

For projects with many phases, gates chain naturally:

```
Phase 1 → Gate 1 → Phase 2 → Gate 2 → Phase 3 → Gate 3 → Complete
```

Each gate only blocks the immediately following phase. A failure at Gate 2 does not affect Phase 1 (already approved).
