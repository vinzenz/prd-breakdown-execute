# Git Commit Format for Task Implementation

## Overview

Each task implementation creates one or more commits in its worktree. The commit history tells the story of implementation and any fixes required.

## Commit Types

### 1. Implementation Commit (Attempt 1)

The first commit contains the initial implementation:

```
[L1-001] Create enums and constants

Implements:
- ProjectStatus enum with values: draft, in_progress, complete
- PersonaType enum with values: developer, designer, pm
- Exported both enums from app/models/__init__.py

Files:
- app/models/enums.py
- app/models/__init__.py
- tests/models/test_enums.py

Task: L1-001
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### 2. Fix Commit (Attempts 2-5)

Each retry creates an additional commit describing what was fixed:

```
[L1-001] Fix: Handle empty string in enum validation

Previous failure:
- test_enum_from_string failed: ValueError on empty input

Fix applied:
- Added empty string check in ProjectStatus.from_string()
- Returns None instead of raising for invalid input

Attempt: 2/5
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### 3. Multiple Fix Commits

A task that required 3 attempts would have this commit history:

```
abc1234 [L1-001] Create enums and constants
def5678 [L1-001] Fix: Handle empty string in enum validation
ghi9012 [L1-001] Fix: Add missing __all__ export
```

## Commit Message Structure

### Implementation Commit Template

```
[{task_id}] {task_name}

Implements:
- {requirement 1 summary}
- {requirement 2 summary}
- {requirement 3 summary}

Files:
- {file 1}
- {file 2}
- {file 3}

Task: {task_id}
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### Fix Commit Template

```
[{task_id}] Fix: {brief description of fix}

Previous failure:
- {error description from verification feedback}

Fix applied:
- {change 1}
- {change 2}

Attempt: {N}/5
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## Commit Guidelines

### Subject Line

- Start with `[{task_id}]` prefix
- For implementations: Use task name from XML
- For fixes: Start with "Fix:" followed by brief description
- Keep under 72 characters
- Use imperative mood ("Create", "Add", "Fix")

### Body

- Blank line after subject
- Describe WHAT was done, not HOW
- List implemented requirements
- List created/modified files
- Include task reference

### Trailer

- Always include `Task: {task_id}` reference
- Always include Co-Authored-By line

## Git Commands

### Creating Implementation Commit

```bash
cd {worktree_path}
git add .
git commit -m "$(cat <<'EOF'
[L1-001] Create enums and constants

Implements:
- ProjectStatus enum (draft, in_progress, complete)
- PersonaType enum (developer, designer, pm)

Files:
- app/models/enums.py
- tests/models/test_enums.py

Task: L1-001
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

### Creating Fix Commit

```bash
cd {worktree_path}
git add .
git commit -m "$(cat <<'EOF'
[L1-001] Fix: Handle empty string in enum validation

Previous failure:
- test_enum_from_string failed: ValueError on empty input

Fix applied:
- Added empty string check in ProjectStatus.from_string()

Attempt: 2/5
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

### Getting Commit Hash

After committing, capture the hash:

```bash
git rev-parse HEAD
# Returns: abc1234def5678...
```

## Commit History Preservation

When the worktree is merged to main, the full commit history is preserved:

```
main:
  │
  ├── Merge worktree-L1-001
  │     │
  │     ├── [L1-001] Create enums and constants
  │     └── [L1-001] Fix: Handle empty string
  │
  └── Merge worktree-L1-002
        │
        └── [L1-002] Create Project model
```

This preserves the narrative of implementation attempts and fixes for future debugging.

## Examples

### Simple Task (1 Attempt)

```
[L1-002] Create Project SQLAlchemy model

Implements:
- Project class with fields: id, name, slug, status, created_at, updated_at
- JSONB column for prd_content
- Unique constraint on slug

Files:
- app/models/project.py
- app/models/__init__.py
- tests/models/test_project.py

Task: L1-002
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### Complex Task (3 Attempts)

**Commit 1:**
```
[L2-003] Create project CRUD API endpoints

Implements:
- POST /api/projects - create project
- GET /api/projects - list projects
- GET /api/projects/{id} - get single project
- PUT /api/projects/{id} - update project
- DELETE /api/projects/{id} - delete project

Files:
- app/api/projects.py
- app/api/__init__.py
- tests/api/test_projects.py

Task: L2-003
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Commit 2:**
```
[L2-003] Fix: Add missing validation for project slug

Previous failure:
- test_create_project_duplicate_slug failed: IntegrityError not caught

Fix applied:
- Added try/except for IntegrityError in create_project
- Return 409 Conflict when slug already exists

Attempt: 2/5
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Commit 3:**
```
[L2-003] Fix: Correct response status code for delete

Previous failure:
- test_delete_project failed: expected 204, got 200

Fix applied:
- Changed return status from 200 to 204 for DELETE endpoint
- Removed response body (204 No Content)

Attempt: 3/5
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```
