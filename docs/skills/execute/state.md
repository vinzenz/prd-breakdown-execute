# State Management

How execution state is tracked and used for resumption.

---

## Overview

Execution state is tracked in `execute-state.json`:

- Location: `{tasks_path}/execute-state.json`
- Created: On first execution
- Updated: After each significant event
- Used: For resume, metrics, debugging

---

## Schema (v2.0)

```json
{
  "schema_version": "2.0",

  "prd_slug": "slack-task-manager",
  "project_path": "/home/user/projects/slack-task-manager",
  "worktree_dir": "/home/user/projects/slack-task-manager/.worktrees",
  "tasks_path": "/home/user/docs/tasks/slack-task-manager",

  "status": "in_progress",
  "current_layer": "2-backend",
  "current_batch": 2,

  "started_at": "2025-01-11T10:00:00Z",
  "updated_at": "2025-01-11T10:15:00Z",
  "completed_at": null,

  "layers": {
    "0-setup": {
      "status": "completed",
      "tasks_total": 4,
      "tasks_completed": 4,
      "tasks_failed": 0,
      "started_at": "2025-01-11T10:00:00Z",
      "completed_at": "2025-01-11T10:03:00Z"
    },
    "1-foundation": {
      "status": "completed",
      "tasks_total": 3,
      "tasks_completed": 3,
      "tasks_failed": 0
    },
    "2-backend": {
      "status": "in_progress",
      "tasks_total": 6,
      "tasks_completed": 4,
      "tasks_failed": 0
    }
  },

  "tasks": {
    "L2-001": {
      "status": "completed",
      "attempts": 1,
      "worktree_path": null,
      "branch": null,
      "commits": [
        {
          "hash": "abc1234",
          "type": "implementation",
          "attempt": 1,
          "created_at": "2025-01-11T10:05:00Z"
        }
      ],
      "merged_at": "2025-01-11T10:06:00Z"
    },
    "L2-002": {
      "status": "completed",
      "attempts": 2,
      "worktree_path": null,
      "branch": null,
      "commits": [
        {"hash": "def5678", "type": "implementation", "attempt": 1},
        {"hash": "ghi9012", "type": "fix", "attempt": 2}
      ],
      "errors": [
        {
          "attempt": 1,
          "message": "test_create_task failed: expected 201 got 422"
        }
      ],
      "retry_feedback": [
        {
          "attempt": 2,
          "feedback": "Add min_length=1 to TaskCreateRequest.title"
        }
      ]
    },
    "L2-003": {
      "status": "in_progress",
      "attempts": 1,
      "worktree_path": "/home/user/projects/slack-task-manager/.worktrees/L2-003",
      "branch": "worktree-L2-003"
    }
  },

  "merge_queue": [
    {"task_id": "L2-003", "priority": 1, "status": "pending"},
    {"task_id": "L2-004", "priority": 2, "status": "pending"}
  ],

  "metrics": {
    "total_tasks": 15,
    "completed_tasks": 11,
    "failed_tasks": 0,
    "total_attempts": 13,
    "total_commits": 14,
    "elapsed_time_seconds": 900
  }
}
```

---

## Status Values

### Global Status

| Status | Description |
|--------|-------------|
| `pending` | Not started |
| `in_progress` | Execution running |
| `completed` | All tasks done |
| `stopped` | Stopped by user |
| `abandoned` | Task failed 5 times |

### Layer Status

| Status | Description |
|--------|-------------|
| `pending` | Layer not started |
| `in_progress` | Tasks being executed |
| `completed` | All tasks done |

### Task Status

| Status | Description |
|--------|-------------|
| `pending` | Not started |
| `in_progress` | Currently executing |
| `verifying` | Awaiting verification |
| `verified` | Passed verification |
| `merging` | Being merged |
| `completed` | Merged to main |
| `failed` | Failed, will retry |
| `abandoned` | Failed 5 times |

---

## Task Lifecycle

```
pending ───► in_progress ───► verifying ───► verified ───► merging ───► completed
                  │               │
                  │               ▼
                  │            failed ◄──────────────────────────────┐
                  │               │                                  │
                  │               └──► (retry if attempts < 5) ──────┘
                  │               │
                  │               └──► abandoned (if attempts = 5)
                  │
                  └──► (on error) ──► failed
```

---

## Merge Queue

Tracks tasks ready for merge:

```json
"merge_queue": [
  {"task_id": "L2-001", "priority": 1, "status": "merged"},
  {"task_id": "L2-003", "priority": 2, "status": "merged"},
  {"task_id": "L2-002", "priority": 3, "status": "ready"},
  {"task_id": "L2-004", "priority": 4, "status": "pending"}
]
```

| Field | Description |
|-------|-------------|
| `task_id` | Task identifier |
| `priority` | Merge order (completion time) |
| `status` | `pending`, `ready`, `merging`, `merged` |

---

## Resume Behavior

When `--resume` is used:

```
Load execute-state.json
    │
    ├── Find current layer
    │
    ├── Identify task statuses:
    │   ├── completed → Skip
    │   ├── abandoned → Skip
    │   ├── failed → Retry (if attempts < 5)
    │   ├── in_progress → Resume
    │   └── pending → Execute
    │
    ├── Rebuild ready queue
    │
    └── Continue execution
```

### Resume Cases

**After normal interruption:**
```
Layer 2: 4/6 completed
Resume: Continue with remaining 2 tasks
```

**After task failure:**
```
L2-003: failed (attempt 3)
Resume: Retry L2-003 (attempt 4)
```

**After abandoned task:**
```
L2-003: abandoned (attempt 5)
Resume: Skip L2-003, stop layer 2
```

---

## Atomic Updates

State is updated atomically:

```python
# 1. Write to temp file
write_file("execute-state.json.tmp", state)

# 2. Atomic rename
rename("execute-state.json.tmp", "execute-state.json")
```

This prevents corruption from:
- Process crashes
- Parallel updates
- Interrupted writes

---

## Concurrency

State tracks parallel execution:

```json
{
  "current_batch": 2,
  "tasks": {
    "L2-003": {"status": "in_progress"},
    "L2-004": {"status": "in_progress"},
    "L2-005": {"status": "in_progress"}
  }
}
```

On completion, each task updates its own status. Merge queue coordinates final ordering.

---

## Metrics

Execution metrics tracked:

```json
"metrics": {
  "total_tasks": 15,
  "completed_tasks": 11,
  "failed_tasks": 0,
  "total_attempts": 13,
  "total_commits": 14,
  "elapsed_time_seconds": 900
}
```

Used for:
- Progress reporting
- Performance analysis
- Debugging

---

## Error Tracking

Errors captured per task:

```json
"errors": [
  {
    "attempt": 1,
    "message": "test_create_task failed",
    "details": "expected 201 got 422",
    "timestamp": "2025-01-11T10:05:00Z"
  }
],
"retry_feedback": [
  {
    "attempt": 2,
    "feedback": "Add min_length=1 validation"
  }
]
```

Enables:
- Post-mortem analysis
- Pattern detection
- Debugging assistance

---

## State Recovery

If state becomes inconsistent:

### 1. Verify Git State

```bash
# Check worktrees exist
git worktree list

# Check branches exist
git branch | grep worktree-
```

### 2. Reconstruct from Git

```bash
# See what commits exist
git log --all --oneline | grep "L2-"
```

### 3. Reset Task

```bash
claude /execute ... --reset-task L2-003 --resume
```

### 4. Full Reset

```bash
# Remove state file
rm docs/tasks/my-project/execute-state.json

# Clean worktrees
git worktree prune
rm -rf .worktrees/

# Start fresh
claude /execute docs/tasks/my-project
```

---

## Best Practices

### 1. Don't Edit Manually

Let the system manage state. Manual edits can cause inconsistencies.

### 2. Backup Before Reset

```bash
cp execute-state.json execute-state.json.bak
```

### 3. Monitor Progress

```bash
# Watch state changes
watch -n 5 'cat execute-state.json | jq .metrics'
```

### 4. Use --dry-run First

```bash
claude /execute ... --dry-run
```

See execution plan without affecting state.

---

## Next Steps

- [Sub-skills Reference](sub-skills.md)
- [Troubleshooting](../../troubleshooting.md)
- [Execute Overview](README.md)
