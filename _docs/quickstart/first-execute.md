---
layout: default
title: Running Execute
parent: Quick Start
nav_order: 3
---

# Running Execute

The `/execute` skill runs your tasks in parallel with TDD, verification, and automatic retries.

## Running Execute

```bash
claude /execute docs/prd/task-management/tasks/
```

## Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        /execute                                  │
├─────────────────────────────────────────────────────────────────┤
│  Layer 0: Setup                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ L0-001-project-init                                          ││
│  │ [████████████████████████████████████████] Complete          ││
│  └─────────────────────────────────────────────────────────────┘│
│                              ↓                                   │
│  Layer 1: Foundation                                             │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐          │
│  │ L1-001-user   │ │ L1-002-proj   │ │ L1-003-task   │          │
│  │ [██████████]  │ │ [██████████]  │ │ [██████████]  │ Parallel │
│  └───────────────┘ └───────────────┘ └───────────────┘          │
│                              ↓                                   │
│  Layer 2: Backend                                                │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐          │
│  │ L2-001-auth   │ │ L2-002-proj   │ │ L2-003-task   │          │
│  │ [████████░░]  │ │ [██████░░░░]  │ │ [████░░░░░░]  │ Parallel │
│  └───────────────┘ └───────────────┘ └───────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

## How It Works

### 1. Git Worktrees

Each task gets its own isolated worktree:

```bash
# Execute creates these automatically
.worktrees/
├── task-L1-001-user-model/     # Isolated copy
├── task-L1-002-project-model/  # Isolated copy
└── task-L1-003-task-model/     # Isolated copy
```

Changes in one worktree don't affect others.

### 2. TDD Workflow

Each task follows test-driven development:

```
┌─────────────────────────────────────────────┐
│ execute-task (in worktree)                  │
│                                             │
│  1. Read task XML                           │
│  2. Write failing tests first               │
│  3. Run tests (expect failure)              │
│  4. Write implementation                    │
│  5. Run tests (expect pass)                 │
│  6. Commit changes                          │
└─────────────────────────────────────────────┘
```

### 3. Independent Verification

A separate agent verifies each task:

```
┌─────────────────────────────────────────────┐
│ execute-verify (fork)                       │
│                                             │
│  - Fresh context (no implementation bias)   │
│  - Runs verification commands               │
│  - Reports pass/fail with details           │
└─────────────────────────────────────────────┘
```

The verifier doesn't see the implementation conversation - only the code.

### 4. Retry on Failure

Failed tasks retry up to 5 times:

```
Attempt 1: FAIL - Tests not passing
  └─ Feedback: "test_login_returns_jwt failed: expected 200, got 401"

Attempt 2: FAIL - Type errors
  └─ Feedback: "mypy: auth.py:45 - Incompatible return type"

Attempt 3: PASS - All verification commands succeed
```

### 5. Sequential Merge

Completed tasks merge to main one at a time:

```
┌─────────────────────────────────────────────┐
│ execute-merge                               │
│                                             │
│  Task L1-001: git merge → main              │
│  Task L1-002: git merge → main              │
│  Task L1-003: git merge → main              │
│                                             │
│  (Sequential to avoid conflicts)            │
└─────────────────────────────────────────────┘
```

## State Tracking

Progress is tracked in `execute-state.json`:

```json
{
  "status": "in_progress",
  "current_layer": 2,
  "started_at": "2025-01-15T10:30:00Z",
  "tasks": {
    "L0-001-project-init": {
      "status": "completed",
      "attempts": 1,
      "completed_at": "2025-01-15T10:31:00Z"
    },
    "L1-001-user-model": {
      "status": "completed",
      "attempts": 1
    },
    "L2-001-auth-api": {
      "status": "in_progress",
      "attempts": 2,
      "last_error": "test_login_returns_jwt failed"
    }
  }
}
```

## Resume Capability

If execution stops (crash, timeout, manual stop):

```bash
# Just run execute again
claude /execute docs/prd/task-management/tasks/
```

It reads `execute-state.json` and continues from where it left off.

## Options

```bash
# Execute specific layers only
claude /execute docs/prd/task-management/tasks/ --layers=1,2

# Dry run (validate without executing)
claude /execute docs/prd/task-management/tasks/ --dry-run

# Resume from specific task
claude /execute docs/prd/task-management/tasks/ --from=L2-001-auth-api
```

## Output

After execution completes:

```
your-project/
├── src/
│   ├── models/
│   │   ├── user.py
│   │   ├── project.py
│   │   └── task.py
│   ├── api/
│   │   ├── auth.py
│   │   ├── projects.py
│   │   └── tasks.py
│   └── main.py
├── tests/
│   ├── models/
│   │   └── ...
│   └── api/
│       └── ...
├── docker-compose.yml
├── Dockerfile
└── pyproject.toml
```

All code is tested and verified.

## Brownfield: Context Finalization

For CRD workflows, execute also updates `PROJECT.md`:

```bash
claude /execute docs/crd/my-change/tasks/
```

After completion:
1. All tasks merged
2. `project-context-finalizer` agent runs
3. `PROJECT.md` updated with new features, APIs, schemas

## Troubleshooting

### Merge Conflicts

If a merge conflict occurs:

1. Execute pauses
2. Manual resolution required
3. Resume with `claude /execute ...`

### Stuck Tasks

If a task fails 5 times:

1. Marked as `failed` in state
2. Execution continues with other tasks
3. Review and fix manually, then resume

### Clean Worktrees

To remove all worktrees:

```bash
# List worktrees
git worktree list

# Remove specific worktree
git worktree remove .worktrees/task-L1-001-user-model

# Remove all
rm -rf .worktrees/
git worktree prune
```
