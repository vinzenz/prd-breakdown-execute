---
layout: default
title: Reference
nav_order: 6
has_children: true
permalink: /docs/reference/
---

# Reference

Technical reference documentation for PRD Breakdown Execute.

## Quick Links

| Reference | Description |
|-----------|-------------|
| [Commands]({{ '/docs/reference/commands/' | relative_url }}) | Full command syntax and options |
| [File Formats]({{ '/docs/reference/file-formats/' | relative_url }}) | XML and JSON schema documentation |

## Command Summary

```bash
# Greenfield workflow
claude /prd                              # Create PRD interactively
claude /breakdown <prd-path>             # Generate tasks
claude /execute <tasks-path>             # Run tasks

# Brownfield workflow
claude /crd                              # Create CRD with context
claude /crd-context                      # Manage PROJECT.md
claude /breakdown <crd-path>             # Generate tasks
claude /execute <tasks-path>             # Run tasks
```

## File Structure Summary

```
project/
├── docs/
│   ├── prd/{slug}/           # PRD documents
│   │   ├── index.md
│   │   ├── features/
│   │   └── tasks/            # Generated tasks
│   │       ├── 0-setup/
│   │       ├── 1-foundation/
│   │       ├── 2-backend/
│   │       ├── manifest.json
│   │       └── execute-state.json
│   └── crd/{slug}/           # CRD documents
│       ├── index.xml
│       └── tasks/
├── PROJECT.md                # Brownfield context
└── .worktrees/               # Parallel execution
```

## State Files

| File | Purpose | Phase |
|------|---------|-------|
| `analysis.json` | PRD/CRD analysis | Breakdown |
| `layer_plan.json` | Layer organization | Breakdown |
| `manifest.json` | Task inventory | Breakdown |
| `execute-state.json` | Execution progress | Execute |
