# Executing Your First Tasks

A detailed walkthrough of the `/execute` command.

---

## Starting Execution

```bash
claude /execute docs/tasks/slack-task-manager
```

You'll see:
```
Starting execution...

Loading manifest from docs/tasks/slack-task-manager/manifest.json
Project: Slack Task Manager
Total tasks: 15
Layers: 5

Creating execution state...
Worktree directory: .worktrees/

Beginning execution...
```

---

## Layer-by-Layer Execution

### Layer 0: Setup

```
╔══════════════════════════════════════════════════════════════════╗
║ Layer 0: Setup                                                   ║
║ Tasks: 4 | Dependencies: None                                    ║
╚══════════════════════════════════════════════════════════════════╝

Batch 1: L0-001, L0-002, L0-003 (parallel, max 3)

  L0-001: Project initialization
    Creating worktree: .worktrees/L0-001
    Reading task spec...
    Writing tests first...
    Implementing...
    Verifying...
    ✓ Verified (attempt 1)
    Added to merge queue

  L0-002: Database setup
    Creating worktree: .worktrees/L0-002
    ...
    ✓ Verified (attempt 1)

  L0-003: Environment configuration
    ...
    ✓ Verified (attempt 1)

Merging (completion order)...
  Merging L0-001 to main... ✓
  Merging L0-003 to main... ✓
  Merging L0-002 to main... ✓

Batch 2: L0-004 (depends on L0-001, L0-002, L0-003)

  L0-004: Initial migration
    ...
    ✓ Verified (attempt 1)

Merging...
  Merging L0-004 to main... ✓

Layer 0 complete: 4/4 tasks
```

---

## Parallel Execution in Action

```
╔══════════════════════════════════════════════════════════════════╗
║ Layer 2: Backend                                                 ║
║ Tasks: 6 | Max parallel: 3                                       ║
╚══════════════════════════════════════════════════════════════════╝

Dependency graph:
  L2-001 (repository) ─┬─► L2-003 (api)
                       │
  L2-002 (service) ────┘

  L2-004 (slack) ──────────► L2-006 (integration)

  L2-005 (auth) ───────────► (layer 4)

Batch 1: L2-001, L2-002, L2-004, L2-005 (4 ready, 3 running)

┌─────────────────────────────────────────────────────────────────┐
│                     Parallel Execution                          │
│                                                                 │
│  L2-001: [████████████████████>          ] Implementing         │
│  L2-002: [██████████████████████████>    ] Verifying            │
│  L2-004: [████████████>                  ] Writing tests        │
│                                                                 │
│  Waiting: L2-005 (max parallel reached)                        │
└─────────────────────────────────────────────────────────────────┘

  L2-002: ✓ Verified (12.3s)

  Starting L2-005...

  L2-001: ✓ Verified (15.1s)
  L2-004: ✓ Verified (18.7s)
  L2-005: ✓ Verified (14.2s)

Merging (completion order)...
  Merging L2-002 to main... ✓
  Merging L2-001 to main... ✓
  Merging L2-004 to main... ✓
  Merging L2-005 to main... ✓

Batch 2: L2-003, L2-006 (dependencies now satisfied)
  ...
```

---

## What Happens Inside Each Task

```
Task L2-003: Task API Endpoints
───────────────────────────────

Step 1: Read task specification
  ├── Parse XML file
  ├── Extract requirements
  └── Identify dependencies

Step 2: Create git worktree
  ├── Branch: worktree-L2-003
  ├── Directory: .worktrees/L2-003
  └── Based on: main

Step 3: Write tests first (TDD)
  ├── Create tests/api/test_tasks.py
  ├── Implement test_create_task()
  ├── Implement test_get_task_not_found()
  └── Run: pytest tests/api/test_tasks.py
      Expected: FAIL (no implementation yet)

Step 4: Implement code
  ├── Create src/api/tasks.py
  ├── Create src/api/schemas/task.py
  ├── Implement all endpoints
  └── Run tests again
      Expected: PASS

Step 5: Git commit
  └── "L2-003: Implement Task API endpoints

       Requirements:
       - R1: Create src/api/tasks.py with CRUD endpoints
       - R2: Create src/api/schemas/task.py with Pydantic models

       Files:
       - src/api/tasks.py
       - src/api/schemas/task.py
       - tests/api/test_tasks.py"

Step 6: Request verification (separate fork)
  └── execute-verify skill (Haiku model)
      Completely independent context
      Cannot see implementation decisions
      Only runs verification commands

Step 7: Handle result
  ├── PASS → Add to merge queue
  └── FAIL → Retry (up to 5 times)
```

---

## Verification

Each task is verified independently:

```
┌─────────────────────────────────────────────────────────────────┐
│                    execute-verify (Haiku)                       │
│                                                                 │
│  Separate context from implementer                              │
│  Cannot see: implementation code, decisions, error history      │
│  Can see: verification commands from task XML                   │
│                                                                 │
│  Step 1: Run pytest tests/api/test_tasks.py -v                  │
│          Result: 5 passed                                       │
│          Status: ✓                                              │
│                                                                 │
│  Step 2: Verify all endpoints respond                           │
│          Result: All 5 endpoints working                        │
│          Status: ✓                                              │
│                                                                 │
│  Step 3: Run python -c "from src.api.tasks import router"       │
│          Result: Import successful                              │
│          Status: ✓                                              │
│                                                                 │
│  Verdict: PASS                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Handling Failures

```
L2-003: Task API Endpoints (attempt 1)
  Writing tests... ✓
  Implementing... ✓
  Verifying...

  ✗ Verification FAILED

  Feedback:
    - Test test_create_task failed
    - Expected status 201, got 422
    - Missing: title validation allows empty string

  Retrying with feedback (attempt 2)...

L2-003: Task API Endpoints (attempt 2)
  Applying fix based on feedback...
  Creating fix commit...
  Verifying...

  ✓ Verified (attempt 2)
```

### Maximum 5 Attempts

If a task fails 5 times:

```
L2-003: Task API Endpoints (attempt 5)
  ...
  ✗ Verification FAILED

  Task ABANDONED after 5 attempts.

  Worktree preserved: .worktrees/L2-003
  You can investigate and fix manually.

  Stopping execution at layer 2-backend.

  To resume after fixing:
    claude /execute docs/tasks/slack-task-manager --resume
```

---

## Sequential Merging

Why merge sequentially when tasks run in parallel?

```
┌─────────────────────────────────────────────────────────────────┐
│                     The Problem                                 │
│                                                                 │
│  L2-001 and L2-002 both add to __init__.py:                     │
│                                                                 │
│  L2-001:                        L2-002:                         │
│    from .task import Task         from .service import TaskService │
│                                                                 │
│  If merged in parallel: CONFLICT                                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     The Solution                                │
│                                                                 │
│  Execution: Parallel              Merge: Sequential             │
│                                                                 │
│  t=0s:  Start L2-001, L2-002                                    │
│  t=12s: L2-002 complete → queue[1]                              │
│  t=15s: L2-001 complete → queue[2]                              │
│                                                                 │
│  t=16s: Merge L2-002 to main                                    │
│  t=17s: Merge L2-001 to main                                    │
│         (automatically rebases if needed)                       │
│                                                                 │
│  No conflicts!                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## State Management

Execution state is tracked in `execute-state.json`:

```json
{
  "schema_version": "2.0",
  "prd_slug": "slack-task-manager",
  "status": "in_progress",
  "current_layer": "2-backend",
  "layers": {
    "0-setup": { "status": "completed", "tasks_completed": 4 },
    "1-foundation": { "status": "completed", "tasks_completed": 3 },
    "2-backend": { "status": "in_progress", "tasks_completed": 4 }
  },
  "tasks": {
    "L2-001": { "status": "completed", "attempts": 1 },
    "L2-002": { "status": "completed", "attempts": 2 },
    "L2-003": { "status": "in_progress", "attempts": 1 }
  },
  "merge_queue": [
    { "task_id": "L2-003", "status": "pending" }
  ]
}
```

---

## Resuming Execution

After interruption:

```bash
claude /execute docs/tasks/slack-task-manager --resume
```

```
Resuming execution...

Loading state from execute-state.json
Status: in_progress at layer 2-backend

Completed:
  - 0-setup: 4/4 tasks
  - 1-foundation: 3/3 tasks
  - 2-backend: 4/6 tasks

Resuming layer 2-backend...
  Pending: L2-005, L2-006

Continuing...
```

---

## Completion

```
╔══════════════════════════════════════════════════════════════════╗
║                     Execution Complete                           ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Project: slack-task-manager                                     ║
║                                                                  ║
║  Layers:                                                         ║
║    0-setup:      4/4 tasks ✓                                     ║
║    1-foundation: 3/3 tasks ✓                                     ║
║    2-backend:    6/6 tasks ✓                                     ║
║    3-frontend:   0/0 tasks ✓ (empty)                             ║
║    4-integration: 2/2 tasks ✓                                    ║
║                                                                  ║
║  Total: 15 tasks completed                                       ║
║  Failed: 0                                                       ║
║  Retried: 3 (L2-002, L2-006, L4-001)                            ║
║                                                                  ║
║  Time: 12m 47s                                                   ║
║                                                                  ║
║  Git commits: 18                                                 ║
║  Files created: 42                                               ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝

Your project is ready at: /path/to/slack-task-manager
```

---

## What You Get

```
slack-task-manager/
├── src/
│   ├── __init__.py
│   ├── main.py              # FastAPI app entry point
│   ├── config.py            # Environment configuration
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py          # SQLAlchemy base
│   │   ├── task.py          # Task model
│   │   ├── user.py          # User model
│   │   └── team.py          # Team model
│   ├── repositories/
│   │   ├── task.py          # Task repository
│   │   └── user.py          # User repository
│   ├── services/
│   │   ├── task.py          # Task service
│   │   └── slack.py         # Slack integration
│   └── api/
│       ├── __init__.py
│       ├── tasks.py         # Task endpoints
│       ├── slack.py         # Slack webhook
│       └── schemas/
│           └── task.py      # Pydantic schemas
├── tests/
│   ├── models/
│   │   └── test_task.py
│   ├── repositories/
│   │   └── test_task.py
│   └── api/
│       └── test_tasks.py
├── alembic/
│   └── versions/
│       └── 001_initial.py
├── .env.example
├── requirements.txt
└── Makefile
```

---

## Troubleshooting

### Worktree conflicts

```bash
# Clean up stale worktrees
git worktree prune

# List worktrees
git worktree list
```

### Task keeps failing

```bash
# Investigate the failed worktree
cd .worktrees/L2-003
git log --oneline
pytest -v

# Fix manually, then resume
cd ../..
claude /execute docs/tasks/slack-task-manager --resume
```

### Reset a specific task

```bash
claude /execute docs/tasks/slack-task-manager \
  --reset-task L2-003 --resume
```

---

## Next Steps

- [Context Fork Deep Dive](../introduction/context-fork.md)
- [Execution Hierarchy](../skills/execute/hierarchy.md)
- [State Management](../skills/execute/state.md)
- [Back to Quick Start](README.md)
