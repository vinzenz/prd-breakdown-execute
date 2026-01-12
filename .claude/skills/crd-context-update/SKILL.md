---
name: crd-context-update
description: Incremental PROJECT.md update using git diff. Only re-analyzes changed areas for efficient context maintenance.
context: fork
agent: crd-context-updater
allowed-tools: Read Write Glob Grep Bash
model: claude-haiku-4-5
---

# Context Update Skill

You perform incremental updates to PROJECT.md by analyzing only what changed since the last context hash.

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` | Yes | Path to project root |

## Prerequisites

- PROJECT.md must exist at `{project_path}/PROJECT.md`
- PROJECT.md must have valid `<last-context-hash>`
- Project must be a git repository

## Update Process

### Step 1: Read Current Context

```bash
cat {project_path}/PROJECT.md
```

Parse and extract:
- `<last-context-hash>` value
- Current `<features>` list
- Current `<api-registry>` entries
- Current `<schema-registry>` entries

### Step 2: Get Changed Files

```bash
git -C {project_path} diff {last_hash}..HEAD --name-only
```

Example output:
```
src/api/theme.py
src/components/ThemeToggle.tsx
src/hooks/useTheme.ts
src/models/user.py
tests/test_theme.py
README.md
```

### Step 3: Categorize Changes

Map changed files to context sections:

| File Pattern | Affects Section |
|--------------|-----------------|
| `*/api/*.py`, `*/routes/*.ts` | `<api-registry>` |
| `*/models/*.py`, `*/models/*.ts` | `<schema-registry>` |
| `*/components/*.tsx`, `*/services/*.py` | `<features>` |
| `*/migrations/*` | `<schema-registry>` |
| `package.json`, `pyproject.toml` | Tech Stack (human section) |

### Step 4: Analyze Changed Sections Only

For each affected section, do targeted analysis:

**If API files changed:**
```bash
# Get the actual changes
git -C {project_path} diff {last_hash}..HEAD -- "*/api/*.py" "*/routes/*.ts"
```

Look for:
- New endpoint decorators/handlers
- Removed endpoints
- Changed signatures

**If model files changed:**
```bash
git -C {project_path} diff {last_hash}..HEAD -- "*/models/*.py" "*/models/*.ts"
```

Look for:
- New model classes
- New fields
- Changed types

**If component/service files changed:**
```bash
git -C {project_path} diff {last_hash}..HEAD -- "*/components/*.tsx" "*/services/*.py"
```

Look for:
- New features (significant new files)
- Modified feature scope

### Step 5: Merge Updates

For each section with changes:

**Adding new entries:**
```xml
<features>
  <!-- existing features unchanged -->
  <feature id="theme" status="complete">
    <name>Theme Support</name>
    <files>src/hooks/useTheme.ts, src/components/ThemeToggle.tsx</files>
  </feature>
</features>
```

**Updating existing entries:**
```xml
<feature id="settings" status="complete">
  <name>User Settings</name>
  <files>src/api/settings.py, src/components/SettingsModal.tsx, src/api/theme.py</files>
  <!-- Added new file to existing feature -->
</feature>
```

**Removing entries (if files deleted):**
- Check if deleted files were the only files for a feature
- If so, remove the feature entry
- If not, update files list

### Step 6: Update Metadata

```bash
# Get current HEAD
git -C {project_path} rev-parse HEAD
```

Update meta section:
```xml
<meta>
  <last-updated>{current ISO timestamp}</last-updated>
  <last-context-hash>{new hash}</last-context-hash>
  <!-- preserve other fields -->
</meta>
```

### Step 7: Write Updated PROJECT.md

Write complete updated file preserving:
- Human-readable sections
- All unchanged context entries
- Proper XML formatting

### Step 8: Report Results

```json
{
  "status": "updated",
  "context_hash": {
    "previous": "{old_hash}",
    "current": "{new_hash}"
  },
  "files_changed": 6,
  "sections_updated": [
    {
      "section": "features",
      "added": 1,
      "modified": 1,
      "removed": 0
    },
    {
      "section": "api-registry",
      "added": 2,
      "modified": 0,
      "removed": 0
    }
  ],
  "summary": "Added theme feature, 2 new API endpoints"
}
```

## Skip Conditions

If only these file types changed, just update the hash:
- Test files only (`tests/`, `__tests__/`)
- Documentation only (`*.md`, `docs/`)
- Build config only (`.github/`, `Makefile`)
- IDE config (`.vscode/`, `.idea/`)

Report:
```json
{
  "status": "hash_only",
  "context_hash": {
    "previous": "{old_hash}",
    "current": "{new_hash}"
  },
  "files_changed": 3,
  "sections_updated": [],
  "summary": "No context-affecting changes. Updated hash only."
}
```

## Error Handling

| Situation | Action |
|-----------|--------|
| Invalid last-context-hash | Report error, suggest `--full` re-scan |
| Hash not in git history | Report error, suggest `--full` re-scan |
| PROJECT.md parse error | Report error, suggest `--full` re-scan |
| Merge conflict in git | Use current state, note in summary |

## Performance

This skill should complete in under 2 minutes for typical changes.
The efficiency comes from:
- Only reading diffs, not full files
- Only analyzing affected sections
- Not re-scanning entire codebase
