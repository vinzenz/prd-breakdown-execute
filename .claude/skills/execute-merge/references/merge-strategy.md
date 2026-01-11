# Merge Strategy

## Overview

Tasks execute in parallel within git worktrees but merge sequentially to main. This ensures:
- No merge conflicts between parallel tasks
- Deterministic, reproducible commit history
- Easy rollback of specific tasks

## Parallel Execution, Sequential Merge

```
Time →

Execution:
  L1-001: [========]                    (10s)
  L1-002: [============]                (15s)
  L1-006: [======]                      (8s)

Merge Queue:
  t=8s:  L1-006 verified → queue (priority 1)
  t=10s: L1-001 verified → queue (priority 2)
  t=15s: L1-002 verified → queue (priority 3)

Merge (sequential):
  t=16s: L1-006 → main (first in queue)
  t=17s: L1-001 → main (second)
  t=18s: L1-002 → main (third)
```

Note: Priority is assigned by completion order, not task ID order.

## Why Sequential Merging?

### The Problem with Parallel Merges

If L1-001 and L1-002 both modify `app/models/__init__.py`:

```python
# L1-001 adds:
from .enums import ProjectStatus

# L1-002 adds:
from .project import Project
```

Parallel merge would cause conflict.

### The Solution

Sequential merge means each task sees the previous task's changes:

1. L1-001 merges: `__init__.py` has `from .enums import ProjectStatus`
2. L1-002 sees this and adds its import below

No conflict possible.

## Merge Queue Structure

```json
{
  "merge_queue": [
    {
      "task_id": "L1-006",
      "priority": 1,
      "status": "merged",
      "merged_at": "2026-01-10T10:16:00Z"
    },
    {
      "task_id": "L1-001",
      "priority": 2,
      "status": "merged",
      "merged_at": "2026-01-10T10:17:00Z"
    },
    {
      "task_id": "L1-002",
      "priority": 3,
      "status": "ready",
      "verified_at": "2026-01-10T10:15:00Z"
    }
  ]
}
```

### Status Values

- `pending`: Task still implementing
- `ready`: Verified, waiting for merge
- `merging`: Currently being merged
- `merged`: Successfully merged

## Merge Commit Format

Use `--no-ff` to create explicit merge commits:

```
*   abc1234 Merge worktree-L1-002: Create Project model
|\
| * def5678 [L1-002] Create Project model
| * ghi9012 [L1-002] Fix: Add missing field
|/
*   jkl3456 Merge worktree-L1-001: Create enums
|\
| * mno7890 [L1-001] Create enums and constants
|/
* pqr1234 Initial commit
```

Benefits:
- Clear task boundaries in git log
- Easy to revert entire task: `git revert -m 1 abc1234`
- Preserves full implementation history

## Worktree Lifecycle

### Creation (by task agent)

```bash
cd {project_path}
git checkout main
git checkout -b worktree-L1-001
git worktree add {worktree_dir}/L1-001 worktree-L1-001
git checkout main
```

### During Implementation

```bash
cd {worktree_dir}/L1-001
# All edits happen here
# Commits created here
```

### After Verification

Worktree added to merge queue with "ready" status.

### Merge

```bash
cd {project_path}
git checkout main
git merge --no-ff worktree-L1-001 -m "..."
```

### Cleanup

```bash
git worktree remove {worktree_dir}/L1-001
git branch -d worktree-L1-001
```

## Conflict Prevention

### Same-Layer Tasks

Tasks in the same batch should not have overlapping files. If they do:

1. Dependency should exist between them
2. They should be in different batches
3. Sequential execution within the batch

### Cross-Layer Tasks

Layer N can safely reference Layer N-1 files because:
- Layer N-1 is fully merged before Layer N starts
- No worktrees from Layer N-1 exist during Layer N

### Shared Files

For files modified by multiple tasks (e.g., `__init__.py`):

1. First task adds its imports/exports
2. Task merges to main
3. Second task starts fresh from main
4. Second task adds its imports/exports
5. No conflict

## Manual Conflict Resolution

If a conflict occurs (shouldn't happen normally):

1. **Merge is aborted**: `git merge --abort`
2. **Worktree preserved**: Full implementation still exists
3. **User notified**: Clear instructions provided

### Resolution Steps

```bash
# In main project
cd {project_path}
git checkout main

# Re-attempt merge
git merge worktree-L1-001

# Resolve conflicts
# Edit conflicting files
git add .
git commit -m "Resolved merge conflict for L1-001"

# Continue with cleanup
git worktree remove {worktree_dir}/L1-001
git branch -d worktree-L1-001
```

### Resume After Resolution

```bash
/execute {tasks_path} --resume
```

The system will see L1-001 as merged (commit exists) and continue.

## Branch Naming Convention

Format: `worktree-{task_id}`

Examples:
- `worktree-L0-001`
- `worktree-L1-001`
- `worktree-L2-003`

Benefits:
- Clear which task owns the branch
- Easy to clean up orphaned branches
- Sortable by layer/task

## Recovery Scenarios

### Orphaned Worktree

If process crashed mid-execution:

```bash
git worktree list
# Shows orphaned worktrees

git worktree prune
# Cleans up stale entries

# For each orphaned worktree:
git worktree remove --force {path}
git branch -D worktree-{task_id}
```

### Orphaned Branch

If worktree removed but branch remains:

```bash
git branch -D worktree-L1-001
```

### State Mismatch

If state says "merging" but no merge in progress:

```bash
# Check git log for merge commit
git log --oneline -10

# If task was merged:
# Update state to "completed"

# If task was not merged:
# Reset state to "verified" and re-run merge
```

## Performance Considerations

### Merge Speed

Each merge is fast (just combining commits):
- Typical merge: < 1 second
- With large files: 1-3 seconds

### Cleanup Speed

Worktree removal is fast:
- Remove worktree: < 1 second
- Delete branch: < 0.1 second

### Bottleneck

The bottleneck is implementation, not merging:
- 3 parallel tasks: ~10-30 min each
- Sequential merge: ~3 seconds each
- Merge overhead: negligible

## Verification Before Merge

Before merging, the system ensures:

1. **Verification passed**: `/execute-verify` returned `all_passed: true`
2. **All commits present**: Changes committed in worktree
3. **No uncommitted changes**: `git status --porcelain` empty
4. **Correct branch**: On `worktree-{task_id}` branch

If any check fails, merge is not attempted.
