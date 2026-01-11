# /execute Skill Reference

Orchestrate parallel task execution with TDD verification.

---

## Overview

The `/execute` skill implements tasks from breakdown using:

- **Parallel execution** in isolated git worktrees
- **TDD workflow** with tests first
- **Independent verification** using separate context
- **Sequential merging** to prevent conflicts
- **State management** for resume capability

```bash
claude /execute docs/tasks/my-project
```

---

## Arguments

| Argument | Description |
|----------|-------------|
| `<tasks-path>` | Path to tasks directory (required) |
| `--resume` | Resume from previous state |
| `--max-parallel <n>` | Maximum concurrent tasks (default: 3) |
| `--layer <name>` | Execute specific layer only |
| `--dry-run` | Preview execution without changes |
| `--reset-task <id>` | Reset specific task for retry |
| `--verbose` | Detailed output |
| `--quiet` | Minimal output (CI mode) |

---

## Architecture

```
/execute (Orchestrator)
    │
    └─► execute-layer (per layer)
            │
            └─► execute-batch (per batch of tasks)
                    │
                    ├─► execute-task ──► execute-verify
                    │       └── git worktree
                    ├─► execute-task ──► execute-verify
                    │       └── git worktree
                    └─► execute-task ──► execute-verify
                            └── git worktree
                    │
                    └─► execute-merge (sequential)
```

All skills run with `context: fork` for isolation.

---

## Execution Flow

### 1. Load and Validate

```
Load manifest.json
Load layer_plan.json
Create/load execute-state.json
Validate task files exist
```

### 2. Layer Execution

```
For each layer (0 → 4):
    │
    ├── Build ready queue (dependencies satisfied)
    │
    ├── Group into batches (max_parallel)
    │
    ├── For each batch:
    │   ├── Launch tasks in parallel
    │   ├── Wait for completion
    │   └── Process merge queue
    │
    └── Mark layer complete
```

### 3. Task Execution

```
For each task:
    │
    ├── Create git worktree
    │
    ├── Write tests first (TDD)
    │
    ├── Implement code
    │
    ├── Create commit
    │
    ├── Request verification (separate fork)
    │
    ├── Handle result:
    │   ├── PASS → Add to merge queue
    │   └── FAIL → Retry (max 5 attempts)
    │
    └── Return result to batch
```

### 4. Merge

```
For each verified task (completion order):
    │
    ├── Switch to main
    │
    ├── Merge worktree branch
    │
    ├── Create merge commit
    │
    └── Clean up worktree
```

---

## Sub-Skills

| Skill | Model | Context | Purpose |
|-------|-------|---------|---------|
| `execute-layer` | sonnet | fork | Layer management |
| `execute-batch` | sonnet | fork | Parallel task launch (uses Task tool) |
| `execute-task` | sonnet | fork | TDD implementation |
| `execute-verify` | haiku | fork | Independent verification |
| `execute-merge` | sonnet | fork | Sequential merge |

[Sub-skill details](sub-skills.md)

---

## Git Worktree Strategy

Each task runs in an isolated git worktree:

```
project/
├── .git/                    # Main repository
├── src/                     # Main branch
└── .worktrees/
    ├── L1-001/              # worktree-L1-001 branch
    │   └── (full project)
    ├── L1-002/              # worktree-L1-002 branch
    │   └── (full project)
    └── L1-003/              # worktree-L1-003 branch
        └── (full project)
```

**Benefits:**
- Complete isolation between tasks
- No git locking conflicts
- Each can build/test independently
- Full commit history preserved

[Worktree details](worktrees.md)

---

## TDD Workflow

Every task follows test-driven development:

```
1. Read task XML
2. Write tests from <test-requirements>
3. Run tests → Should FAIL (red)
4. Implement code from <requirements>
5. Run tests → Should PASS (green)
6. Commit implementation
7. Request verification
```

[TDD details](tdd.md)

---

## Verification

Independent verification in separate context:

```
execute-task context              execute-verify context
        │                                  │
        │ Implements...                    │
        │                                  │
        └─── "Please verify" ─────────────►│
             (task XML only)               │
                                           │
             Does NOT send:                ▼
             - Implementation code    Fresh context
             - Design decisions       Only sees:
             - Error history          - Verification commands
                                      - Expected outcomes

                                      Returns:
                                      PASS or FAIL + feedback
```

This prevents bias from implementation context.

---

## State Management

Execution state tracked in `execute-state.json`:

```json
{
  "schema_version": "2.0",
  "status": "in_progress",
  "current_layer": "2-backend",

  "layers": {
    "0-setup": {"status": "completed", "tasks_completed": 4},
    "1-foundation": {"status": "completed", "tasks_completed": 3},
    "2-backend": {"status": "in_progress", "tasks_completed": 2}
  },

  "tasks": {
    "L2-001": {"status": "completed", "attempts": 1},
    "L2-002": {"status": "in_progress", "attempts": 2}
  },

  "merge_queue": [
    {"task_id": "L2-003", "status": "pending"}
  ]
}
```

[State schema details](state.md)

---

## Error Handling

### Task Failure

```
Task fails verification
        │
        ▼
    Attempt < 5?
    ├── YES: Retry with feedback
    └── NO:  Abandon task, stop layer
```

### Abandoned Tasks

When a task fails 5 times:

1. Worktree preserved for debugging
2. Execution stops at current layer
3. User can fix manually and resume

### Resume After Failure

```bash
# Fix the issue in .worktrees/L2-003/
# Then resume:
claude /execute docs/tasks/my-project --resume
```

### Reset Specific Task

```bash
claude /execute docs/tasks/my-project \
  --reset-task L2-003 --resume
```

---

## Parallel Execution

```
Batch with 3 tasks, max_parallel=3:

t=0s:   Launch L1-001, L1-002, L1-003
        │
t=8s:   L1-003 complete → merge queue
t=10s:  L1-001 complete → merge queue
t=15s:  L1-002 complete → merge queue
        │
t=16s:  Merge L1-003 to main
t=17s:  Merge L1-001 to main
t=18s:  Merge L1-002 to main
```

Tasks run in parallel, merge sequentially.

---

## Common Options

### Dry Run

```bash
claude /execute docs/tasks/my-project --dry-run
```

Shows execution plan without making changes.

### Increase Parallelism

```bash
claude /execute docs/tasks/my-project --max-parallel 5
```

Run up to 5 tasks simultaneously.

### Single Layer

```bash
claude /execute docs/tasks/my-project --layer 2-backend
```

Execute only the specified layer.

### Verbose Output

```bash
claude /execute docs/tasks/my-project --verbose
```

Show detailed progress for each task.

---

## Output

On completion:

```
╔════════════════════════════════════════════════════════════╗
║                   Execution Complete                       ║
╠════════════════════════════════════════════════════════════╣
║  Tasks completed: 15                                       ║
║  Tasks failed: 0                                           ║
║  Total time: 12m 47s                                       ║
║                                                            ║
║  Git commits: 18                                           ║
║  Files created: 42                                         ║
╚════════════════════════════════════════════════════════════╝
```

---

## Troubleshooting

### Worktree conflicts

```bash
git worktree prune
git worktree list
```

### Stuck merge queue

Check `execute-state.json` for queue status.

### Verification keeps failing

Check `.worktrees/{task-id}/` to debug the implementation.

[Full troubleshooting guide](../../troubleshooting.md)

---

## Next Steps

- [Execution Hierarchy](hierarchy.md)
- [Git Worktrees](worktrees.md)
- [TDD Workflow](tdd.md)
- [State Management](state.md)
- [Sub-skills Reference](sub-skills.md)
