# Execute State Schema

## Overview

The execution state is stored in `execute-state.json` in the tasks directory. This file tracks progress across all layers and tasks, enabling resume functionality and progress reporting.

## File Location

```
{tasks-path}/execute-state.json
```

Example: `docs/tasks/voice-prd-generator/execute-state.json`

## Schema Version

Current version: `2.0`

## Complete Schema

```json
{
  "schema_version": "2.0",
  "prd_slug": "voice-prd-generator",
  "project_path": "/home/user/projects/my-app",
  "worktree_dir": "/home/user/projects/.worktrees",
  "tasks_path": "/home/user/docs/tasks/voice-prd-generator",

  "started_at": "2026-01-10T10:00:00Z",
  "updated_at": "2026-01-10T12:30:00Z",
  "completed_at": null,

  "status": "in_progress",
  "current_layer": "2-backend",
  "current_batch": 2,

  "options": {
    "max_parallel": 3,
    "layer_filter": null,
    "task_filter": null,
    "commit_prefix": "",
    "no_commits": false,
    "verbose": false,
    "quiet": false
  },

  "layers": {
    "0-setup": {
      "status": "completed",
      "started_at": "2026-01-10T10:00:00Z",
      "completed_at": "2026-01-10T10:30:00Z",
      "tasks_total": 4,
      "tasks_completed": 4,
      "tasks_failed": 0
    },
    "1-foundation": {
      "status": "completed",
      "started_at": "2026-01-10T10:30:00Z",
      "completed_at": "2026-01-10T11:00:00Z",
      "tasks_total": 6,
      "tasks_completed": 6,
      "tasks_failed": 0
    },
    "2-backend": {
      "status": "in_progress",
      "started_at": "2026-01-10T11:00:00Z",
      "completed_at": null,
      "tasks_total": 9,
      "tasks_completed": 5,
      "tasks_failed": 0
    }
  },

  "tasks": {
    "L1-001": {
      "status": "completed",
      "attempts": 1,
      "worktree_path": null,
      "branch": null,
      "started_at": "2026-01-10T10:30:00Z",
      "completed_at": "2026-01-10T10:35:00Z",
      "merged_at": "2026-01-10T10:36:00Z",
      "commits": [
        {
          "hash": "abc1234",
          "type": "implementation",
          "attempt": 1,
          "message": "[L1-001] Create enums and constants"
        }
      ],
      "errors": [],
      "retry_feedback": []
    },
    "L2-003": {
      "status": "in_progress",
      "attempts": 2,
      "worktree_path": "/home/user/projects/.worktrees/L2-003",
      "branch": "worktree-L2-003",
      "started_at": "2026-01-10T11:30:00Z",
      "completed_at": null,
      "merged_at": null,
      "commits": [
        {
          "hash": "def5678",
          "type": "implementation",
          "attempt": 1,
          "message": "[L2-003] Create project CRUD API"
        },
        {
          "hash": "ghi9012",
          "type": "fix",
          "attempt": 2,
          "message": "[L2-003] Fix: Handle duplicate slug error",
          "fixed": "Added try/except for IntegrityError"
        }
      ],
      "errors": [
        {
          "attempt": 1,
          "type": "verification_failed",
          "step": "pytest tests/api/test_projects.py",
          "message": "test_create_duplicate_slug failed",
          "timestamp": "2026-01-10T11:35:00Z"
        }
      ],
      "retry_feedback": [
        {
          "attempt": 2,
          "feedback": "Add IntegrityError handling in create_project endpoint"
        }
      ]
    }
  },

  "worktrees": {
    "L2-003": {
      "task_id": "L2-003",
      "path": "/home/user/projects/.worktrees/L2-003",
      "branch": "worktree-L2-003",
      "created_at": "2026-01-10T11:30:00Z",
      "status": "active"
    },
    "L2-004": {
      "task_id": "L2-004",
      "path": "/home/user/projects/.worktrees/L2-004",
      "branch": "worktree-L2-004",
      "created_at": "2026-01-10T11:30:00Z",
      "status": "active"
    }
  },

  "merge_queue": [
    {"task_id": "L2-001", "priority": 1, "status": "merged"},
    {"task_id": "L2-002", "priority": 2, "status": "merged"},
    {"task_id": "L2-003", "priority": 3, "status": "pending"},
    {"task_id": "L2-004", "priority": 4, "status": "pending"}
  ],

  "completed": ["L0-001", "L0-002", "L0-003", "L0-004", "L1-001", "L1-002", "L1-003", "L1-004", "L1-005", "L1-006", "L2-001", "L2-002"],
  "failed": [],
  "abandoned": [],

  "metrics": {
    "tasks_total": 48,
    "tasks_completed": 12,
    "tasks_failed": 0,
    "tasks_abandoned": 0,
    "tasks_remaining": 36,
    "total_attempts": 13,
    "total_retries": 1,
    "elapsed_seconds": 9000
  },

  "context_update": {
    "status": "pending",
    "project_md_path": null,
    "features_added": [],
    "endpoints_added": [],
    "models_added": [],
    "commit_hash": null
  }
}
```

## Field Descriptions

### Root Fields

| Field | Type | Description |
|-------|------|-------------|
| `schema_version` | string | Schema version for compatibility |
| `prd_slug` | string | PRD identifier from manifest |
| `project_path` | string | Target project directory |
| `worktree_dir` | string | Directory containing worktrees |
| `tasks_path` | string | Directory containing task XMLs |
| `started_at` | ISO8601 | When execution started |
| `updated_at` | ISO8601 | Last state update |
| `completed_at` | ISO8601 | When execution completed (null if in progress) |
| `status` | enum | Overall status |
| `current_layer` | string | Currently executing layer |
| `current_batch` | number | Current batch number within layer |

### Status Values

**Overall status:**
- `initializing` - Setting up execution
- `in_progress` - Executing tasks
- `completed` - All tasks done
- `stopped` - User-initiated stop
- `abandoned` - Task hit max retries

**Layer status:**
- `pending` - Not started
- `in_progress` - Executing
- `completed` - All tasks done
- `blocked` - Dependent task abandoned

**Task status:**
- `pending` - Not started
- `in_progress` - Being implemented
- `verifying` - Verification running
- `verified` - Passed verification
- `merging` - Being merged to main
- `completed` - Merged successfully
- `failed` - Failed attempt (will retry)
- `abandoned` - Max retries reached

**Worktree status:**
- `creating` - Being created
- `active` - In use
- `merging` - Branch being merged
- `cleaned` - Removed after merge
- `abandoned` - Preserved for debugging

**Merge queue status:**
- `pending` - Waiting for verification
- `ready` - Verified, ready to merge
- `merging` - Currently merging
- `merged` - Successfully merged

### Options

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_parallel` | number | 3 | Max concurrent tasks |
| `layer_filter` | string | null | Only execute this layer |
| `task_filter` | string | null | Only execute this task |
| `commit_prefix` | string | "" | Prefix for commit messages |
| `no_commits` | boolean | false | Skip git commits |
| `verbose` | boolean | false | Detailed output |
| `quiet` | boolean | false | Minimal output |

### Task Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | enum | Current task status |
| `attempts` | number | Number of attempts so far |
| `worktree_path` | string | Path to worktree (null if completed) |
| `branch` | string | Git branch name |
| `started_at` | ISO8601 | When task started |
| `completed_at` | ISO8601 | When task completed |
| `merged_at` | ISO8601 | When merged to main |
| `commits` | array | List of commits in worktree |
| `errors` | array | Errors from failed attempts |
| `retry_feedback` | array | Feedback for retries |

### Commit Object

```json
{
  "hash": "abc1234",
  "type": "implementation|fix",
  "attempt": 1,
  "message": "[L1-001] Create enums",
  "fixed": "Description of fix (for fix commits only)"
}
```

### Error Object

```json
{
  "attempt": 1,
  "type": "verification_failed|implementation_error|merge_conflict",
  "step": "pytest tests/...",
  "message": "Detailed error message",
  "timestamp": "2026-01-10T11:35:00Z"
}
```

### Retry Feedback Object

```json
{
  "attempt": 2,
  "feedback": "Actionable fix suggestion from verification"
}
```

### Context Update Object

For CRD-based projects with PROJECT.md, this tracks the post-execution context update:

```json
{
  "status": "pending|in_progress|completed|skipped",
  "project_md_path": "/path/to/PROJECT.md",
  "features_added": ["dark-mode", "theme-settings"],
  "endpoints_added": ["/api/settings/theme"],
  "models_added": ["ThemePreference"],
  "commit_hash": "abc123"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `status` | enum | pending, in_progress, completed, skipped (no PROJECT.md) |
| `project_md_path` | string | Path to PROJECT.md (null if not exists) |
| `features_added` | array | Feature IDs added to PROJECT.md |
| `endpoints_added` | array | API endpoints added to api-registry |
| `models_added` | array | Model names added to schema-registry |
| `commit_hash` | string | Git commit hash of context update |

## State Transitions

### Task Lifecycle

```
pending
    │
    v
in_progress (attempt 1)
    │
    v
verifying
    │
    ├── PASS ──> verified ──> merging ──> completed
    │
    └── FAIL ──> failed
                    │
                    v (attempt < 5)
                in_progress (attempt N)
                    │
                    v
                verifying
                    │
                    ├── PASS ──> verified ──> merging ──> completed
                    │
                    └── FAIL ──> (repeat or abandoned if attempt >= 5)
```

### Layer Lifecycle

```
pending
    │
    v
in_progress
    │
    ├── all tasks completed ──> completed
    │
    └── any task abandoned ──> blocked
```

## State File Operations

### Reading State

```python
import json
from pathlib import Path

def load_state(tasks_path: str) -> dict:
    state_file = Path(tasks_path) / "execute-state.json"
    if state_file.exists():
        return json.loads(state_file.read_text())
    return None
```

### Writing State

```python
def save_state(tasks_path: str, state: dict):
    state["updated_at"] = datetime.utcnow().isoformat() + "Z"
    state_file = Path(tasks_path) / "execute-state.json"
    state_file.write_text(json.dumps(state, indent=2))
```

### Initializing State

```python
def init_state(prd_slug: str, project_path: str, tasks_path: str, options: dict) -> dict:
    return {
        "schema_version": "2.0",
        "prd_slug": prd_slug,
        "project_path": project_path,
        "worktree_dir": str(Path(project_path).parent / ".worktrees"),
        "tasks_path": tasks_path,
        "started_at": datetime.utcnow().isoformat() + "Z",
        "updated_at": datetime.utcnow().isoformat() + "Z",
        "completed_at": None,
        "status": "initializing",
        "current_layer": None,
        "current_batch": None,
        "options": options,
        "layers": {},
        "tasks": {},
        "worktrees": {},
        "merge_queue": [],
        "completed": [],
        "failed": [],
        "abandoned": [],
        "metrics": {
            "tasks_total": 0,
            "tasks_completed": 0,
            "tasks_failed": 0,
            "tasks_abandoned": 0,
            "tasks_remaining": 0,
            "total_attempts": 0,
            "total_retries": 0,
            "elapsed_seconds": 0
        }
    }
```

## Resume Behavior

When `--resume` is specified:

1. Load existing state file
2. Re-evaluate ready queue based on current state
3. For `in_progress` tasks:
   - If worktree exists: continue from current attempt
   - If worktree missing: reset to pending
4. Skip `completed` tasks
5. Retry `failed` tasks (if attempts < 5)
6. Skip `abandoned` tasks

## Concurrency Considerations

Multiple agents may update state concurrently. Use atomic writes:

```bash
# Write to temp file, then rename (atomic on most filesystems)
echo "$STATE_JSON" > execute-state.json.tmp
mv execute-state.json.tmp execute-state.json
```

For critical sections (merge queue updates), consider file locking:

```bash
# Simple lock file approach
while [ -f execute-state.lock ]; do sleep 0.1; done
touch execute-state.lock
# ... update state ...
rm execute-state.lock
```
