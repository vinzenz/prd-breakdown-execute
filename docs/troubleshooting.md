# Troubleshooting

Common issues and solutions.

---

## PRD Issues

### PRD won't complete

**Symptom:** PRD creation stalls or loops on a phase.

**Solutions:**

1. **Be more specific**
   ```
   Bad:  "I want a good app"
   Good: "A REST API for task management with CRUD operations"
   ```

2. **Resume and skip**
   ```bash
   claude /prd --resume
   # When prompted, choose to skip optional sections
   ```

3. **Check what-next.md**
   ```bash
   cat docs/prd/my-project/what-next.md
   # Shows what's incomplete
   ```

### Can't find PRD to resume

**Symptom:** `--resume` shows no incomplete PRDs.

**Solutions:**

1. **Check directory**
   ```bash
   ls docs/prd/
   ```

2. **Check status in index.md**
   ```bash
   grep "<status>" docs/prd/*/index.md
   # Should show "in-progress" for resumable PRDs
   ```

---

## Breakdown Issues

### Review keeps failing

**Symptom:** Task generation fails review repeatedly.

**Solutions:**

1. **Check PRD for vague language**
   ```xml
   <!-- Bad -->
   <description>Handle things appropriately</description>

   <!-- Good -->
   <description>
     Validate input: title max 200 chars, required.
     Return 422 on validation failure.
   </description>
   ```

2. **Add concrete acceptance criteria**
   ```xml
   <criterion>
     <given>User submits empty title</given>
     <when>POST /api/tasks with {"title": ""}</when>
     <then>Response is 422 with error details</then>
   </criterion>
   ```

3. **Check analysis.json**
   ```bash
   cat docs/tasks/my-project/analysis.json | jq
   # Verify inferred models and endpoints are correct
   ```

### Wrong template detected

**Symptom:** Layer 0 uses wrong template.

**Solutions:**

1. **Edit analysis.json**
   ```bash
   # Change template field
   vim docs/tasks/my-project/analysis.json
   ```

2. **Re-run with skip-analysis**
   ```bash
   claude /breakdown docs/prd/my-project/index.md --skip-analysis
   ```

### Too many files per task

**Symptom:** Tasks have more than 3 files.

**Solution:** Re-run breakdown - tasks should auto-split.

---

## Execution Issues

### Task keeps failing

**Symptom:** Task fails verification repeatedly.

**Solutions:**

1. **Check the worktree**
   ```bash
   cd .worktrees/L2-003
   cat tests/api/test_tasks.py
   pytest tests/api/test_tasks.py -v
   ```

2. **Check verification feedback**
   ```bash
   cat docs/tasks/my-project/execute-state.json | jq '.tasks["L2-003"]'
   ```

3. **Fix manually and resume**
   ```bash
   cd .worktrees/L2-003
   # Make fixes
   git add . && git commit -m "Manual fix"
   cd ../..
   claude /execute docs/tasks/my-project --resume
   ```

### Worktree conflicts

**Symptom:** Git errors about worktrees.

**Solutions:**

1. **Prune stale worktrees**
   ```bash
   git worktree prune
   ```

2. **List and clean**
   ```bash
   git worktree list
   git worktree remove .worktrees/L2-003 --force
   ```

3. **Delete branches**
   ```bash
   git branch -D worktree-L2-003
   ```

### Can't resume

**Symptom:** Resume fails or starts from beginning.

**Solutions:**

1. **Check state file exists**
   ```bash
   ls docs/tasks/my-project/execute-state.json
   ```

2. **Validate state file**
   ```bash
   cat docs/tasks/my-project/execute-state.json | jq '.'
   ```

3. **Reset specific task**
   ```bash
   claude /execute docs/tasks/my-project --reset-task L2-003 --resume
   ```

### Merge conflicts

**Symptom:** Merge to main fails.

**Solutions:**

1. **Check branch state**
   ```bash
   cd .worktrees/L2-003
   git status
   git log --oneline -5
   ```

2. **Manual merge**
   ```bash
   git checkout main
   git merge worktree-L2-003
   # Resolve conflicts
   git commit
   ```

3. **Update state**
   ```bash
   # Edit execute-state.json to mark task complete
   ```

---

## Git Issues

### Worktree locked

**Symptom:** Can't create or remove worktree.

**Solution:**

```bash
git worktree unlock .worktrees/L2-003
git worktree remove .worktrees/L2-003
```

### Branch already exists

**Symptom:** Can't create worktree branch.

**Solution:**

```bash
git branch -D worktree-L2-003
claude /execute docs/tasks/my-project --reset-task L2-003 --resume
```

### Detached HEAD

**Symptom:** Worktree is in detached HEAD state.

**Solution:**

```bash
cd .worktrees/L2-003
git checkout worktree-L2-003
```

---

## State Recovery

### Corrupted state file

**Symptom:** JSON parse error in state file.

**Solutions:**

1. **Restore from backup**
   ```bash
   cp execute-state.json.bak execute-state.json
   ```

2. **Reconstruct from git**
   ```bash
   git log --all --oneline | grep "L[0-9]-"
   # Manually update state based on commits
   ```

3. **Start fresh**
   ```bash
   rm execute-state.json
   rm -rf .worktrees/
   git worktree prune
   claude /execute docs/tasks/my-project
   ```

### Inconsistent state

**Symptom:** State doesn't match actual progress.

**Solutions:**

1. **Compare with git**
   ```bash
   git log --oneline | head -20
   # Check which tasks were actually merged
   ```

2. **Reset and resume**
   ```bash
   claude /execute docs/tasks/my-project --resume
   # System will re-evaluate state
   ```

---

## Performance Issues

### Execution too slow

**Solutions:**

1. **Increase parallelism**
   ```bash
   claude /execute docs/tasks/my-project --max-parallel 5
   ```

2. **Check resource usage**
   ```bash
   htop
   # Watch for memory/CPU constraints
   ```

### Context window errors

**Symptom:** Model complains about context size.

**Solutions:**

1. **Tasks may be too large**
   - Check task XML size
   - Consider splitting PRD features

2. **Reduce dependencies**
   - Check if all interface contracts are necessary
   - Simplify if possible

---

## Quick Fixes

### Reset everything

```bash
# Remove all execution artifacts
rm docs/tasks/my-project/execute-state.json
rm -rf .worktrees/
git worktree prune
git branch | grep worktree- | xargs git branch -D

# Start fresh
claude /execute docs/tasks/my-project
```

### Check current status

```bash
# Overall status
cat docs/tasks/my-project/execute-state.json | jq '{status, current_layer, metrics}'

# Task statuses
cat docs/tasks/my-project/execute-state.json | jq '.tasks | to_entries[] | {id: .key, status: .value.status, attempts: .value.attempts}'
```

### Verbose debugging

```bash
claude /execute docs/tasks/my-project --verbose
```

---

## Getting Help

If issues persist:

1. Check [Architecture](../ARCHITECTURE.md) for system design
2. Review [State Management](skills/execute/state.md)
3. Examine task XML for specification issues
4. Report issues at: https://github.com/anthropics/claude-code/issues

---

## Next Steps

- [Execute Reference](skills/execute/README.md)
- [State Management](skills/execute/state.md)
- [Commands Reference](reference/commands.md)
