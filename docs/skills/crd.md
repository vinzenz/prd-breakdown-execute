# /crd - Change Request Documents

The `/crd` command creates focused change request documents for existing codebases. Unlike PRDs which define new products, CRDs target specific changes to brownfield projects.

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         /crd Workflow                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Context      2. Capture      3. Impact       4. Generate        │
│     Check           Change          Analysis         CRD            │
│                                                                      │
│  PROJECT.md  →  Change type   →  Affected     →  docs/crd/          │
│  generated      Motivation       files           {slug}.md          │
│  or updated     Requirements     Features                           │
│                                  APIs                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Usage

```bash
# Create a CRD with description
/crd --project /path/to/my-app "Add dark mode toggle"
/crd -p /path/to/my-app "Remove deprecated API endpoints"

# List existing CRDs
/crd -p /path/to/my-app --list

# Check CRD status
/crd -p /path/to/my-app --status dark-mode-toggle
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` or `-p <path>` | **Yes** | Path to target project |
| Description text | No | Initial description of the change |
| `--list` | No | List existing CRDs for the project |
| `--status <slug>` | No | Check status of a specific CRD |

---

## Context Management

The CRD workflow automatically manages project context through `PROJECT.md`.

### First Run (No PROJECT.md)

When you run `/crd` on a project for the first time:

1. **Deep Investigation** - Analyzes your entire codebase
2. **Tech Stack Detection** - Identifies frameworks, languages, patterns
3. **Feature Inventory** - Maps existing functionality
4. **API Extraction** - Documents endpoints and schemas
5. **PROJECT.md Generation** - Creates comprehensive context file

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
```

### Subsequent Runs (PROJECT.md Exists)

When you run `/crd` again:

1. **Hash Check** - Compares `<last-context-hash>` with current `git HEAD`
2. **If Stale** - Runs incremental update via `git diff`
3. **If Current** - Proceeds directly to change capture

```
Checking context status...
Context is stale (12 commits behind)

Updating context...
  Files analyzed: 8
  Sections updated: api-registry, features

New context hash: abc123def456
```

---

## Change Types

CRD supports four change types:

| Type | Indicators | Description |
|------|------------|-------------|
| `feature-add` | "add", "create", "new" | Adding new functionality |
| `feature-modify` | "change", "update", "enhance" | Modifying existing features |
| `feature-remove` | "remove", "delete", "deprecate" | Removing functionality |
| `refactor` | "refactor", "restructure" | Code reorganization |

The command automatically classifies your change and asks for confirmation.

---

## Impact Analysis

After capturing your change request, CRD runs automatic impact analysis:

```
## Impact Analysis

**Files to Modify:**
- src/components/SettingsModal.tsx
- src/api/settings.py

**Files to Create:**
- src/hooks/useTheme.ts

**Affected Features:**
- settings (User Settings)

**Affected APIs:**
- PUT /api/settings (add theme field)

**Breaking Changes:** None

**Scope:** Small (3 files)
**Confidence:** High
```

You can review and adjust this analysis before generating the CRD.

---

## Output Format

CRD documents are written to `docs/crd/{slug}.md` in XML format:

```xml
<crd>
  <meta>
    <name>Add Dark Mode Toggle</name>
    <slug>dark-mode-toggle</slug>
    <type>feature-add</type>
    <created>2026-01-12</created>
    <status>ready</status>
  </meta>

  <context>
    <project-ref>PROJECT.md</project-ref>
    <related-features>
      <feature-ref id="settings">User Settings (modified)</feature-ref>
    </related-features>
  </context>

  <change-request>
    <summary>Add dark mode toggle to settings</summary>
    <motivation>User accessibility request</motivation>
  </change-request>

  <impact-analysis>
    <affected-files>
      <file action="modify">src/components/SettingsModal.tsx</file>
      <file action="create">src/hooks/useTheme.ts</file>
    </affected-files>
    <affected-features>
      <feature id="settings">Add theme toggle</feature>
    </affected-features>
    <breaking-changes>none</breaking-changes>
    <scope>small</scope>
  </impact-analysis>

  <requirements>
    <requirement id="1" priority="must-have">
      Toggle switch in settings modal
    </requirement>
  </requirements>

  <acceptance-criteria>
    <criterion id="1">
      <given>User on settings page</given>
      <when>User toggles dark mode</when>
      <then>UI switches to dark theme</then>
    </criterion>
  </acceptance-criteria>
</crd>
```

---

## Next Steps

After creating a CRD, run `/breakdown` to generate implementation tasks:

```bash
/breakdown docs/crd/dark-mode-toggle.md --project-path /path/to/my-app
```

Then execute with `/execute`:

```bash
/execute docs/tasks/dark-mode-toggle --project-path /path/to/my-app
```

After execution completes, `PROJECT.md` is automatically updated with new features.

---

## /crd-context

Standalone command for PROJECT.md management without creating a CRD.

### Usage

```bash
# Generate or update PROJECT.md
/crd-context --project /path/to/my-app

# Force full re-investigation
/crd-context -p /path/to/my-app --full

# Check if context is stale
/crd-context -p /path/to/my-app --check

# Show changes since last update
/crd-context -p /path/to/my-app --diff
```

### Arguments

| Argument | Description |
|----------|-------------|
| `--project <path>` | Target project path (required) |
| `--full` | Force full re-investigation |
| `--check` | Check staleness without updating |
| `--diff` | Show changes since last update |

---

## Sub-Skills

| Skill | Purpose |
|-------|---------|
| `crd-investigate` | Deep codebase analysis for PROJECT.md |
| `crd-context-update` | Incremental context update via git diff |
| `crd-impact-analysis` | Analyze change impact on existing code |
| `project-context-finalizer` | Update PROJECT.md after /execute |

---

## PROJECT.md Format

PROJECT.md contains both human-readable and machine-readable sections:

```markdown
# Project: My App

## Overview
Brief description of the project.

## Architecture
Tech stack, patterns, component structure.

## Context Metadata
<!-- Machine-readable section -->
<project-context version="1.0">
  <meta>
    <last-updated>2026-01-12T10:30:00Z</last-updated>
    <last-context-hash>abc123def456</last-context-hash>
  </meta>

  <features>
    <feature id="auth" status="complete">
      <name>User Authentication</name>
      <files>src/api/auth.py, src/models/user.py</files>
    </feature>
  </features>

  <api-registry>
    <endpoint method="POST" path="/api/auth/login">
      <request>{ email: string, password: string }</request>
      <response>{ token: string, user: User }</response>
    </endpoint>
  </api-registry>

  <schema-registry>
    <model name="User" table="users">
      <field name="id" type="uuid" primary="true"/>
      <field name="email" type="string" unique="true"/>
    </model>
  </schema-registry>
</project-context>
```

---

## Comparison: PRD vs CRD

| Aspect | PRD | CRD |
|--------|-----|-----|
| Use case | New projects | Existing projects |
| Context | None (greenfield) | PROJECT.md |
| Layers | 5 (0-4) | 2-3 (based on impact) |
| Scope | Full product | Focused change |
| Setup | Layer 0 included | No Layer 0 |
| Post-execute | None | PROJECT.md updated |
