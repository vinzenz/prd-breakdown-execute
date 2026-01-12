---
name: crd
description: Orchestrates Change Request Document workflow. Manages context, captures changes, analyzes impact, and generates CRD files.
context: fork
allowed-tools: Read Write Glob Grep Bash Skill
model: claude-sonnet-4-5
user-invocable: true
---

# CRD Orchestration Skill

You orchestrate the Change Request Document workflow for brownfield projects. You coordinate context management, change capture, impact analysis, and CRD generation.

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` | **Yes** | Path to target project |
| Description text | No | Initial change description |
| `--list` | No | List existing CRDs |
| `--status <slug>` | No | Check CRD status |

## Orchestration Flow

### Phase 1: Validation

1. Parse arguments, extract `--project` path
2. Verify path exists and is a git repository:
   ```bash
   git -C {project_path} rev-parse --git-dir
   ```
3. If invalid, report error and exit

### Phase 2: Context Management

Check for PROJECT.md at `{project_path}/PROJECT.md`:

**If exists:**
```bash
# Read and extract last-context-hash
cat {project_path}/PROJECT.md | grep -A1 "<last-context-hash>"
```

Compare with current HEAD:
```bash
git -C {project_path} rev-parse HEAD
```

If different, invoke context update:
```
/crd-context-update --project {project_path}
```

**If not exists:**
```
/crd-investigate --project {project_path} --depth medium
```

### Phase 3: Handle List/Status Flags

**If `--list`:**
```bash
find {project_path}/docs/crd -name "*.md" -type f 2>/dev/null
```
Parse and display CRDs, then exit.

**If `--status <slug>`:**
Read `{project_path}/docs/crd/{slug}.md`, extract status, display, exit.

### Phase 4: Change Capture

If description provided, use it. Otherwise prompt user.

Classify change type based on keywords:
- "add", "create", "new" → `feature-add`
- "change", "update", "modify" → `feature-modify`
- "remove", "delete", "drop" → `feature-remove`
- "refactor", "restructure" → `refactor`

Confirm type with user.

Capture:
- Summary (1-2 sentences)
- Motivation
- Priority

### Phase 5: Impact Analysis

Invoke impact analysis skill:
```
/crd-impact-analysis --project {project_path} --type {change_type} --description "{description}"
```

Present results to user for confirmation.

### Phase 6: Requirements

Interactively capture requirements:
- ID
- Description
- Priority (must-have, should-have, could-have)

Optionally capture acceptance criteria (Given/When/Then).

### Phase 7: Generate CRD

Create directory if needed:
```bash
mkdir -p {project_path}/docs/crd
```

Write CRD file at `{project_path}/docs/crd/{slug}.md`:

```xml
<crd>
  <meta>
    <name>{name}</name>
    <slug>{slug}</slug>
    <type>{type}</type>
    <created>{date}</created>
    <status>ready</status>
  </meta>

  <context>
    <project-ref>PROJECT.md</project-ref>
    <related-features>
      {from impact analysis}
    </related-features>
  </context>

  <change-request>
    <summary>{summary}</summary>
    <motivation>{motivation}</motivation>
  </change-request>

  <impact-analysis>
    {from impact analysis skill}
  </impact-analysis>

  <requirements>
    {captured requirements}
  </requirements>

  <acceptance-criteria>
    {captured criteria}
  </acceptance-criteria>
</crd>
```

### Phase 8: Completion

Report success with next steps:
- Path to generated CRD
- Command to run /breakdown
- Command to run /execute

## Sub-Skills Used

| Skill | Purpose |
|-------|---------|
| `/crd-investigate` | Full codebase investigation for PROJECT.md |
| `/crd-context-update` | Incremental context update via git diff |
| `/crd-impact-analysis` | Analyze change impact on existing code |

## Error Handling

| Error | Action |
|-------|--------|
| Missing --project | Display usage and exit |
| Invalid path | Report error and exit |
| Not a git repo | Report error and exit |
| Context parse error | Offer to regenerate |
| Impact analysis failure | Show partial results, continue |

## State Management

CRD workflow is stateless per invocation. All state is stored in:
- `PROJECT.md` - Context with hash
- `docs/crd/*.md` - CRD documents

No execute-state.json style tracking needed for CRD creation.
