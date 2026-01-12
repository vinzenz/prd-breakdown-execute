---
name: crd-investigate
description: Deep codebase investigation to generate PROJECT.md context. Analyzes architecture, patterns, features, APIs, and schemas.
context: fork
agent: crd-investigator
allowed-tools: Read Glob Grep Bash
model: claude-sonnet-4-5
---

# Codebase Investigation Skill

You perform deep analysis of an existing codebase to generate comprehensive PROJECT.md context.

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` | Yes | Path to project root |
| `--depth <level>` | No | Investigation depth: `quick`, `medium` (default), `deep` |

## Investigation Process

### Step 1: Project Structure

```bash
# Get current git hash for context tracking
git -C {project_path} rev-parse HEAD

# List top-level structure
ls -la {project_path}

# Get directory tree (excluding common ignores)
find {project_path} -type d \
  -not -path '*/\.*' \
  -not -path '*/node_modules/*' \
  -not -path '*/venv/*' \
  -not -path '*/.venv/*' \
  -not -path '*/__pycache__/*' \
  -not -path '*/dist/*' \
  -not -path '*/build/*' \
  | head -100
```

### Step 2: Tech Stack Detection

Check for package/config files:

```bash
# Check multiple patterns
ls {project_path}/pyproject.toml {project_path}/requirements.txt {project_path}/setup.py 2>/dev/null
ls {project_path}/package.json {project_path}/tsconfig.json 2>/dev/null
ls {project_path}/go.mod {project_path}/Cargo.toml 2>/dev/null
```

Read package files to extract:
- Language and version
- Framework (FastAPI, React, Go Chi, etc.)
- Database ORM (SQLAlchemy, Drizzle, GORM)
- Key dependencies

### Step 3: Feature Discovery

Based on detected framework, look for features:

**FastAPI/Python:**
```bash
grep -r "@router\." {project_path}/src --include="*.py" -l
grep -r "@app\." {project_path}/src --include="*.py" -l
```

**Express/Node:**
```bash
grep -r "router\." {project_path}/src --include="*.ts" --include="*.js" -l
```

**React:**
```bash
find {project_path}/src -name "*.tsx" -path "*/components/*" -o -name "*.tsx" -path "*/pages/*"
```

### Step 4: API Endpoint Extraction

**FastAPI:**
```python
# Look for patterns like:
@router.get("/users")
@router.post("/auth/login")
```

Read files and extract:
- Method (GET, POST, PUT, DELETE)
- Path
- Request/response types from type hints

**Express/TanStack:**
```typescript
// Look for patterns like:
router.get('/users', handler)
app.post('/auth/login', handler)
```

### Step 5: Schema/Model Extraction

**SQLAlchemy:**
```python
# Look for class definitions inheriting from Base
class User(Base):
    __tablename__ = "users"
```

**Drizzle:**
```typescript
// Look for table definitions
export const users = pgTable('users', {...})
```

Extract:
- Model name
- Table name
- Fields with types
- Relationships (foreign keys)

### Step 6: Pattern Analysis

Identify common patterns:
- File naming conventions
- Import patterns
- Error handling approach
- Authentication mechanism
- State management (frontend)

### Step 7: Generate PROJECT.md

Create PROJECT.md at `{project_path}/PROJECT.md`:

```markdown
# Project: {detected name}

## Overview
{Inferred from README.md or code structure}

## Architecture

### Tech Stack
{Detected stack}

### Component Structure
{Directory tree with annotations}

### Key Patterns
{Detected patterns}

## Context Metadata
<project-context version="1.0">
  <meta>
    <last-updated>{now}</last-updated>
    <last-context-hash>{git hash}</last-context-hash>
  </meta>

  <features>
    {Discovered features}
  </features>

  <api-registry>
    {Extracted endpoints}
  </api-registry>

  <schema-registry>
    {Extracted models}
  </schema-registry>
</project-context>
```

### Step 8: Create CRD Directory

```bash
mkdir -p {project_path}/docs/crd
```

## Depth Levels

### Quick (5-10 minutes)
- Project structure
- Tech stack from package files
- Main entry points
- README overview

### Medium (15-20 minutes) - Default
- All of Quick
- Feature inventory from routes/components
- API endpoint extraction
- Database model extraction
- Basic pattern identification

### Deep (30+ minutes)
- All of Medium
- Full dependency graph
- Test coverage analysis
- Integration point mapping
- Detailed pattern analysis
- Code quality assessment

## Output

After completion, report:

```
Investigation complete.

PROJECT.md generated at: {project_path}/PROJECT.md

Summary:
- Tech Stack: {stack}
- Features: {count} identified
- API Endpoints: {count} documented
- Database Models: {count} cataloged
- Context Hash: {hash}

docs/crd/ directory created for future CRDs.
```

## Error Handling

| Situation | Action |
|-----------|--------|
| Empty project | Create minimal PROJECT.md |
| No recognizable framework | Document structure only |
| Binary files | Skip, note in overview |
| Permission errors | Report and continue with accessible files |
| Very large codebase | Limit scope, note truncation |
