# /crd-context - Project Context Management

Manage PROJECT.md context for CRD workflows. Use this to generate, update, or check project context status.

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` or `-p <path>` | **Yes** | Path to target project |
| `--full` | No | Force full re-investigation (ignores existing context) |
| `--check` | No | Check if context is stale (don't update) |
| `--diff` | No | Show what changed since last context update |

## Examples

```bash
# Generate or update PROJECT.md
/crd-context --project /path/to/my-app

# Force full re-scan
/crd-context -p /path/to/my-app --full

# Check if context is stale
/crd-context -p /path/to/my-app --check

# Show changes since last update
/crd-context -p /path/to/my-app --diff
```

## Workflow

### 1. Validate Project Path

1. Check that `--project` or `-p` is provided
2. Verify the path exists and is a git repository
3. Set working context to the project path

### 2. Handle `--check` Flag

If `--check` is present:

1. Read PROJECT.md and extract `last-context-hash`
2. Get current `git rev-parse HEAD`
3. Compare:

**If same:**
```
Context is up-to-date.
  PROJECT.md: {path}
  Context hash: {hash} (current)
  Last updated: {timestamp}
```

**If different:**
```
Context is STALE.
  PROJECT.md: {path}
  Context hash: {old_hash}
  Current HEAD: {new_hash}
  Files changed: {count}

Run `/crd-context -p {path}` to update.
```

Exit after check.

### 3. Handle `--diff` Flag

If `--diff` is present:

1. Read PROJECT.md and extract `last-context-hash`
2. Run `git diff {last-context-hash}..HEAD --name-only`
3. Categorize changes by impact area:

```
Changes since last context update ({old_hash}..{new_hash}):

## Files Changed: 12

### API Changes (affects api-registry)
  M src/api/settings.py
  A src/api/theme.py

### Model Changes (affects schema-registry)
  M src/models/user.py

### Component Changes (affects features)
  A src/components/ThemeToggle.tsx
  M src/components/SettingsModal.tsx

### Other Changes (no context impact)
  M tests/test_settings.py
  M README.md
```

Exit after showing diff.

### 4. Handle `--full` Flag or Missing PROJECT.md

If `--full` is present OR PROJECT.md doesn't exist:

Invoke the `/crd-investigate` skill:

```
/crd-investigate --project {project_path} --depth medium
```

Report completion:

```
Generated PROJECT.md with full codebase context.
  Path: {project_path}/PROJECT.md
  Features found: {count}
  API endpoints: {count}
  Database models: {count}
  Context hash: {hash}
```

### 5. Incremental Update (Default)

If PROJECT.md exists and `--full` is not present:

1. Check if context is stale (compare hashes)
2. If same: Report "Context is up-to-date" and exit
3. If different: Invoke `/crd-context-update` skill

```
/crd-context-update --project {project_path}
```

Report completion:

```
Updated PROJECT.md with incremental changes.
  Files analyzed: {count}
  Sections updated: {list}
  New context hash: {hash}
```

## PROJECT.md Location

PROJECT.md is always at: `{project_path}/PROJECT.md`

If creating for the first time, also create:
- `{project_path}/docs/crd/` directory (for future CRDs)

## Error Handling

| Error | Response |
|-------|----------|
| Missing `--project` | "The `--project` (or `-p`) argument is required." |
| Invalid project path | "The path `{path}` doesn't exist or isn't a directory." |
| Not a git repo | "The project at `{path}` is not a git repository." |
| Corrupt PROJECT.md | Offer to regenerate with `--full` |
| Invalid context hash | Offer to regenerate with `--full` |

## Output Examples

### Fresh Generation

```
Investigating codebase at /path/to/my-app...

Tech Stack Detected:
  - Backend: FastAPI (Python 3.11)
  - Frontend: React + Vite
  - Database: PostgreSQL with SQLAlchemy

Features Found:
  - auth: User Authentication (complete)
  - settings: User Settings (complete)
  - dashboard: Dashboard (complete)

API Endpoints: 12
Database Models: 5

Generated PROJECT.md at /path/to/my-app/PROJECT.md
Context hash: abc123def456
```

### Incremental Update

```
Checking context status...

Context is stale:
  Previous hash: abc123
  Current HEAD: def456
  Files changed: 8

Updating context...

Updated sections:
  - api-registry: +2 endpoints
  - features: +1 feature (dark-mode)

Updated PROJECT.md
New context hash: def456
```

### Up-to-date Check

```
Context is up-to-date.

PROJECT.md: /path/to/my-app/PROJECT.md
Context hash: def456 (current)
Last updated: 2026-01-12T10:30:00Z

Features: 4
API Endpoints: 14
Database Models: 5
```
