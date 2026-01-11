# Git Worktree Isolation

How tasks execute in isolated git worktrees.

---

## Overview

Each task runs in its own git worktree - a separate working directory with its own branch, allowing true parallel execution without conflicts.

```
project/
├── .git/                    # Main repository
├── src/                     # Main branch
│
└── .worktrees/              # Parallel workspaces
    ├── L1-001/              # worktree-L1-001 branch
    │   ├── src/
    │   ├── tests/
    │   └── ...
    │
    ├── L1-002/              # worktree-L1-002 branch
    │   ├── src/
    │   ├── tests/
    │   └── ...
    │
    └── L1-003/              # worktree-L1-003 branch
        ├── src/
        ├── tests/
        └── ...
```

---

## Why Worktrees?

### Problem: Git Locking

Without worktrees, parallel execution fails:

```
Task 1: git checkout -b task-1
Task 2: git checkout -b task-2
        └── FAILS: Cannot switch branch while Task 1 is working

Task 1: Editing src/models/user.py
Task 2: Editing src/models/user.py
        └── CONFLICT: Same file modified
```

### Solution: Complete Isolation

With worktrees:

```
Task 1: Working in .worktrees/L1-001/
        └── Has own src/models/user.py

Task 2: Working in .worktrees/L1-002/
        └── Has own src/models/user.py

No conflicts. No locking. True parallel.
```

---

## Worktree Lifecycle

### 1. Creation

When a task starts (first attempt):

```bash
# Create branch and worktree
git worktree add .worktrees/L1-001 -b worktree-L1-001 main
```

This creates:
- New branch `worktree-L1-001` from `main`
- Directory `.worktrees/L1-001/` with full project copy
- Linked to main repository's `.git/`

### 2. Work

Task executes in the worktree:

```
cd .worktrees/L1-001

# Write tests
edit tests/models/test_user.py

# Run tests (should fail)
pytest tests/models/test_user.py

# Implement
edit src/models/user.py

# Run tests (should pass)
pytest tests/models/test_user.py

# Commit
git add .
git commit -m "L1-001: Implement User model"
```

### 3. Retry

On verification failure (attempt 2+):

```
# Reuse existing worktree
cd .worktrees/L1-001

# Apply fix based on feedback
edit src/models/user.py

# Commit fix
git add .
git commit -m "L1-001: Fix password hash validation"

# Re-verify
```

### 4. Merge

On verification success:

```bash
# From main project directory
git checkout main
git merge worktree-L1-001 --no-ff -m "Merge worktree-L1-001: User model"
```

### 5. Cleanup

After successful merge:

```bash
# Remove worktree
git worktree remove .worktrees/L1-001

# Delete branch
git branch -d worktree-L1-001
```

---

## Parallel Execution

Three tasks running simultaneously:

```
t=0s:   Create worktrees (parallel)
        ├── git worktree add .worktrees/L1-001 ...
        ├── git worktree add .worktrees/L1-002 ...
        └── git worktree add .worktrees/L1-003 ...

t=1s:   All tasks start working (parallel)
        ├── L1-001: Writing tests...
        ├── L1-002: Writing tests...
        └── L1-003: Writing tests...

t=5s:   ├── L1-001: Implementing...
        ├── L1-002: Implementing...
        └── L1-003: Implementing...

t=8s:   └── L1-003: Complete! (verified)
t=10s:  ├── L1-001: Complete! (verified)
t=15s:  └── L1-002: Complete! (verified)
```

Each worktree:
- Has its own branch
- Can compile independently
- Can run tests independently
- Commits don't affect others

---

## Branch Naming

Convention: `worktree-{task-id}`

```
worktree-L0-001
worktree-L1-001
worktree-L1-002
worktree-L2-003
...
```

This makes it easy to:
- Identify which branch belongs to which task
- Clean up stale branches
- Debug failed tasks

---

## Sequential Merging

**Why sequential?**

Multiple tasks might modify the same file:

```
L1-001 adds: from .user import User
L1-002 adds: from .team import Team
L1-003 adds: from .project import Project

All modify: src/models/__init__.py
```

If merged in parallel: CONFLICT

**Solution:**

```
Execute: Parallel     Merge: Sequential
        │                    │
        ├── L1-001 ─────────►│ Merge L1-001
        ├── L1-002 ─────────►│ Merge L1-002 (rebased if needed)
        └── L1-003 ─────────►│ Merge L1-003 (rebased if needed)
```

Merge order = completion order. Each merge builds on previous.

---

## Worktree Preservation

**Failed tasks preserve their worktree:**

```
L2-003 fails verification (attempt 5)
    │
    └── .worktrees/L2-003/ preserved
        ├── Contains failed implementation
        ├── All commit history present
        └── User can debug manually
```

This enables:
- Post-mortem debugging
- Manual fixes
- Resume after human intervention

**Successful tasks remove their worktree:**

```
L2-001 verified and merged
    │
    └── .worktrees/L2-001/ removed
        └── Branch deleted
```

---

## State Tracking

Worktree state in `execute-state.json`:

```json
{
  "tasks": {
    "L1-001": {
      "status": "in_progress",
      "worktree_path": "/project/.worktrees/L1-001",
      "branch": "worktree-L1-001",
      "commits": [
        {"hash": "abc123", "type": "implementation", "attempt": 1}
      ]
    }
  }
}
```

---

## Common Issues

### Stale Worktrees

After crashes or interruptions:

```bash
# List worktrees
git worktree list

# Prune stale entries
git worktree prune
```

### Locked Worktrees

If a worktree is "locked":

```bash
# Unlock
git worktree unlock .worktrees/L1-001

# Or remove forcefully
git worktree remove --force .worktrees/L1-001
```

### Branch Conflicts

If branch already exists:

```bash
# Remove old branch
git branch -D worktree-L1-001

# Retry task
claude /execute ... --reset-task L1-001 --resume
```

---

## Best Practices

### 1. Use Absolute Paths

Worktrees work best with absolute paths internally.

### 2. Don't Manually Modify

Let the system manage worktrees. Manual changes can corrupt state.

### 3. Clean Up After Failures

```bash
# After fixing a failed task manually
git worktree remove .worktrees/L2-003
git branch -d worktree-L2-003
```

### 4. Check Status

```bash
# See all worktrees
git worktree list

# See worktree-specific branches
git branch | grep worktree-
```

---

## Next Steps

- [TDD Workflow](tdd.md)
- [State Management](state.md)
- [Execute Overview](README.md)
