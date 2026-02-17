# Dependency Waves

Within each phase, tasks form a directed acyclic graph (DAG) based on their dependencies. Execute them in waves: each wave contains tasks whose dependencies are all satisfied.

## Algorithm

```
completed = []
pending = [all phase tasks]

while pending is not empty:
    wave = [t for t in pending
            where t.blockedBy ⊆ completed]

    if wave is empty:
        error: circular dependency or unresolvable blocker
        break

    dispatch all tasks in wave (parallel via Task tool)
    wait for wave to complete
    move completed wave tasks from pending to completed

gate review activates (all phase tasks complete)
```

## CC Task Implementation

CC's `addBlockedBy` encodes the DAG. The manager reads the task list and builds waves:

```
# Read all phase tasks
tasks = TaskList filtered by "[Phase N:" prefix

# Find Wave 1: tasks with no blockers (or all blockers completed)
wave_1 = [t for t in tasks
          where t.blockedBy is empty
          or all items in t.blockedBy are completed]

# Dispatch Wave 1
for task in wave_1:
    TaskUpdate: task.id, status=in_progress
    dispatch worker via Task tool

# After Wave 1 completes, find Wave 2
wave_2 = [t for t in remaining tasks
          where all items in t.blockedBy are completed]

# Continue until no pending tasks remain
```

## Example

```
Phase 2: API
├── Task A: Define API schema            (no deps)       → Wave 1
├── Task B: Set up routing               (no deps)       → Wave 1
├── Task C: Implement auth endpoints     (depends: A, B) → Wave 2
├── Task D: Implement CRUD endpoints     (depends: A, B) → Wave 2
└── Task E: Write integration tests      (depends: C, D) → Wave 3

Wave 1: [A, B]  — parallel
Wave 2: [C, D]  — parallel (after A, B complete)
Wave 3: [E]     — sequential (after C, D complete)
```

## Dispatch Pattern

For each task in a wave, dispatch a worker using `orchestrating-work`:

```
Task tool (general-purpose):
  description: "Coder: [task subject]"
  prompt: [worker template with task context]
```

Independent tasks within a wave dispatch as parallel Task tool calls. The manager waits for all wave tasks to return before building the next wave.

## Handling Failures

| Worker status | Manager action                                              |
| ------------- | ----------------------------------------------------------- |
| `complete`    | Mark CC task completed; include in next wave's resolved set |
| `partial`     | Record progress; re-queue task in next wave                 |
| `incapable`   | Reassign to different worker or escalate to architect       |
| `blocked`     | Record blocker; continue other wave tasks; surface to user  |

A single task failure does not halt the wave. Other independent tasks continue. Only tasks that depend on the failed task are affected.
