# Execute Sub-Skills

Reference for the component skills used by `/execute`.

---

## Overview

The execute skill orchestrates 5 sub-skills:

```
/execute (orchestrator)
     │
     ├──► execute-layer (per layer)
     │         │
     │         ├──► execute-batch (per batch)
     │         │         │
     │         │         └──► execute-task (parallel, via Task tool)
     │         │                   │
     │         │                   └──► execute-verify
     │         │
     │         └──► execute-merge (sequential)
     │
     └──► (repeat for each layer)
```

All sub-skills run with `context: fork`.

---

## execute-layer

**Purpose:** Manage execution of a single architectural layer.

**Context:** fork
**Model:** sonnet

### Input

- Layer identifier (e.g., "2-backend")
- State file path
- Tasks path
- Max parallel setting

### Process

1. Load layer plan and dependency graph
2. Load current task states
3. Build ready queue (tasks with satisfied dependencies)
4. While ready tasks exist:
   - Group into batch (up to max_parallel)
   - Fork execute-batch
   - Wait for batch completion
   - Process merge queue
   - Re-evaluate ready queue
5. Mark layer complete

### Ready Queue Algorithm

```
For each pending task in layer:
    │
    ├── Get task dependencies from layer_plan.json
    │
    ├── Check if all dependencies are "completed"
    │
    └── If yes: Add to ready queue

Ready queue sorted by:
    1. Priority (from task meta)
    2. Task ID (alphabetical)

Take min(ready_queue.size, max_parallel) for next batch
```

### Output

Layer completion status:
- All tasks completed → layer complete
- Task abandoned → layer stopped

---

## execute-batch

**Purpose:** Launch multiple tasks in parallel using the Task tool.

**Context:** fork
**Model:** sonnet

### Input

- List of task IDs to execute
- Tasks path
- State file path

### Process

1. Load task XML for each task
2. **In SINGLE message with multiple Task tool calls:**
   - Launch execute-task for each (run_in_background: true)
3. For each launched task:
   - Wait with TaskOutput (timeout: 10 minutes)
   - Collect result
4. Update state with results
5. Add verified tasks to merge queue
6. Return batch result

### Critical: Parallel Launch

All tasks MUST be launched in a single message:

```
Message contains:
├── Task(execute-task, task="L2-001", run_in_background=true)
├── Task(execute-task, task="L2-002", run_in_background=true)
└── Task(execute-task, task="L2-003", run_in_background=true)
```

This ensures true parallel execution, not sequential.

### Output

```json
{
  "completed": ["L2-001", "L2-002"],
  "failed": ["L2-003"],
  "abandoned": []
}
```

---

## execute-task

**Purpose:** Implement a single task using TDD in a git worktree.

**Context:** fork
**Model:** sonnet

### Input

- Task XML file path
- Worktree directory
- Attempt number

### Process

1. Parse task XML
2. Setup worktree:
   - First attempt: Create new worktree from main
   - Retry: Reuse existing worktree
3. Write tests from `<test-requirements>`
4. Run tests (expect failure for new code)
5. Implement code from `<requirements>`
6. Run tests (expect pass)
7. Create git commit
8. Fork execute-verify for independent verification
9. Handle verification result:
   - PASS: Return verified status
   - FAIL (attempts < 5): Apply feedback, commit fix, retry verify
   - FAIL (attempt = 5): Return abandoned status

### Commit Format

Implementation:
```
L2-003: Implement Task API endpoints

Requirements:
- R1: Create src/api/tasks.py
- R2: Create src/api/schemas/task.py

Files:
- src/api/tasks.py
- src/api/schemas/task.py
- tests/api/test_tasks.py
```

Fix:
```
L2-003: Fix title validation

Feedback: test_create_task_empty_title failed
Fix: Added min_length=1 to TaskCreateRequest

Attempt: 2
```

### Output

```json
{
  "task_id": "L2-003",
  "status": "verified" | "failed" | "abandoned",
  "attempts": 2,
  "commits": ["abc123", "def456"]
}
```

---

## execute-verify

**Purpose:** Independent verification using separate context.

**Context:** fork
**Model:** haiku (fast, focused)

### Input

- Task XML (NOT implementation code)
- Worktree path (for running commands)

### Critical: No Implementation Context

The verifier receives only:
- Task XML with verification steps
- Path to worktree for command execution

Does NOT receive:
- Implementation code
- Design decisions
- Error history
- Previous attempts

This ensures unbiased verification.

### Process

1. Parse `<verification>` section from task XML
2. For each verification step:
   - Run commands
   - Check expected outcomes
   - Record pass/fail
3. If any step fails:
   - Generate actionable feedback
   - Specific to the failure
   - Suggests concrete fix

### Verification Step Types

| Type | Example | Check |
|------|---------|-------|
| `Run:` | `pytest tests/api/test_tasks.py` | Exit code 0 |
| `Verify:` | `All 5 tests pass` | Output contains expected |
| `Check:` | `Router has 5 routes` | Condition true |

### Output

Pass:
```json
{
  "verdict": "PASS",
  "steps": [
    {"step": "pytest ...", "status": "PASS"},
    {"step": "All tests pass", "status": "PASS"}
  ]
}
```

Fail:
```json
{
  "verdict": "FAIL",
  "steps": [
    {"step": "pytest ...", "status": "FAIL"}
  ],
  "feedback": "test_create_task_empty_title failed: expected 422 got 201. Add min_length=1 validation to TaskCreateRequest.title field."
}
```

### Feedback Quality

Good feedback is:
- Specific: Which test, which assertion
- Actionable: What to change
- Concrete: Exact values, exact locations

```
Good:
  "test_create_task_empty_title failed: expected 422 got 201.
   Add min_length=1 to TaskCreateRequest.title in
   src/api/schemas/task.py line 5"

Bad:
  "Test failed."
```

---

## execute-merge

**Purpose:** Merge completed task worktrees to main branch.

**Context:** fork
**Model:** sonnet

### Input

- Task ID
- Worktree path
- Branch name

### Process

1. Verify worktree state:
   - On correct branch
   - All changes committed
   - No uncommitted files
2. Switch to main branch
3. Pull latest (if remote configured)
4. Merge worktree branch:
   - `git merge --no-ff worktree-L2-003`
   - Creates explicit merge commit
5. Remove worktree:
   - `git worktree remove .worktrees/L2-003`
6. Delete branch:
   - `git branch -d worktree-L2-003`
7. Update state

### Merge Commit Format

```
Merge worktree-L2-003: Task API endpoints

Layer: 2-backend
Task: L2-003
Commits: 2 (1 implementation + 1 fix)

Implements:
- Task CRUD API endpoints
- Pydantic request/response schemas

Verification: Passed (attempt 2)
```

### Sequential Execution

Merges happen one at a time:
- Prevents merge conflicts
- Maintains clean history
- Each merge builds on previous

Order: Task completion time (first completed = first merged)

### Output

```json
{
  "task_id": "L2-003",
  "merged": true,
  "merge_commit": "xyz789"
}
```

---

## Skill Coordination

### Data Flow

```
execute
    │
    │ layer="2-backend", max_parallel=3
    ▼
execute-layer
    │
    │ batch=[L2-001, L2-002, L2-003]
    ▼
execute-batch
    │
    ├──► Task(execute-task, L2-001)
    ├──► Task(execute-task, L2-002)
    └──► Task(execute-task, L2-003)
    │
    │ results: [verified, verified, failed]
    ▼
execute-layer
    │
    │ merge_queue: [L2-001, L2-002]
    │
    ├──► execute-merge(L2-001)
    └──► execute-merge(L2-002)
    │
    │ Next batch: [L2-003] (retry)
    ▼
execute-batch
    │
    └──► Task(execute-task, L2-003, attempt=2)
```

### State Updates

Each skill updates state at completion:

| Skill | Updates |
|-------|---------|
| execute-batch | Task statuses, errors, retry feedback |
| execute-merge | Task completion, merge queue status |
| execute-layer | Layer status, completion time |
| execute | Global status, metrics |

---

## Next Steps

- [Execution Hierarchy](hierarchy.md)
- [State Management](state.md)
- [Execute Overview](README.md)
