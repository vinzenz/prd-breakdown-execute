---
layout: default
title: Architecture
parent: Introduction
nav_order: 3
---

# Architecture

PRD Breakdown Execute is built on a layered skill architecture that orchestrates AI agents for autonomous software development.

## System Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                            │
│  /prd  │  /crd  │  /breakdown  │  /execute  │  /crd-context      │
└────────┴────────┴──────────────┴────────────┴────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                      SKILL ORCHESTRATION                          │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐      │
│  │   breakdown    │  │    execute     │  │      crd       │      │
│  │   (fork)       │  │   (fork)       │  │    (fork)      │      │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘      │
└──────────┼───────────────────┼───────────────────┼───────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌──────────────────────────────────────────────────────────────────┐
│                       SUB-SKILLS LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │
│  │ analyze-prd  │  │ execute-task │  │ investigate  │            │
│  │ plan-layers  │  │ execute-     │  │ impact-      │            │
│  │ generate-    │  │   verify     │  │   analysis   │            │
│  │   tasks      │  │ execute-     │  │ context-     │            │
│  │ review-tasks │  │   merge      │  │   update     │            │
│  └──────────────┘  └──────────────┘  └──────────────┘            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                        AGENT LAYER                                │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐         │
│  │task-generator │  │task-implement │  │crd-investigat │         │
│  │task-reviewer  │  │verification-  │  │crd-context-   │         │
│  │               │  │   runner      │  │   updater     │         │
│  └───────────────┘  └───────────────┘  └───────────────┘         │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                      EXECUTION LAYER                              │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │                    Git Worktrees                          │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │    │
│  │  │worktree │  │worktree │  │worktree │  │worktree │      │    │
│  │  │  task-1 │  │  task-2 │  │  task-3 │  │  task-n │      │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

## Component Types

### Commands

User-facing entry points in `.claude/commands/`:

| Command | Purpose |
|---------|---------|
| `/prd` | Interactive PRD creation (8 phases) |
| `/crd` | Change request with context analysis |
| `/crd-context` | Standalone PROJECT.md management |

Commands are markdown files with frontmatter that define the interface.

### Skills

Workflow components in `.claude/skills/`:

```
.claude/skills/breakdown/
├── SKILL.md              # Skill definition
└── references/           # Reference files
    ├── layer-definitions.md
    └── task-format-spec.md
```

Skills define:
- **Context mode**: `fork` or default (shared)
- **Model**: `sonnet`, `haiku`, or `opus`
- **Tools**: Allowed tool list
- **Instructions**: What the skill does

### Agents

Specialized executors in `.claude/agents/`:

| Agent | Purpose |
|-------|---------|
| `task-generator` | Creates task XML files |
| `task-implementer` | Executes tasks with TDD |
| `task-reviewer` | Validates task quality |
| `verification-runner` | Runs verification commands |
| `crd-investigator` | Deep codebase analysis |
| `crd-context-updater` | Incremental context updates |

Agents are invoked via the Task tool with `subagent_type`.

## Data Flow

### Greenfield Workflow

```
User Input
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /prd - Interactive PRD Creation                             │
│   Phase 1: Capture idea                                     │
│   Phase 2: Choose tech stack                                │
│   Phase 3: Define features                                  │
│   Phase 4-8: Dependencies, review, output                   │
│   Output: docs/prd/{slug}/index.md + features/              │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /breakdown - Task Generation                                │
│   ┌──────────────┐                                          │
│   │ analyze-prd  │ → analysis.json                         │
│   └──────┬───────┘                                          │
│          ▼                                                  │
│   ┌──────────────┐                                          │
│   │ plan-layers  │ → layer_plan.json                       │
│   └──────┬───────┘                                          │
│          ▼                                                  │
│   ┌──────────────┐                                          │
│   │generate-tasks│ → tasks/{layer}/*.xml                   │
│   └──────┬───────┘                                          │
│          ▼                                                  │
│   ┌──────────────┐                                          │
│   │ review-tasks │ → validated or retry                    │
│   └──────────────┘                                          │
│   Output: tasks/ directory with XML files                   │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /execute - Parallel Implementation                          │
│   For each layer (0 → 4):                                  │
│     ┌──────────────┐                                        │
│     │execute-layer │                                        │
│     └──────┬───────┘                                        │
│            ▼                                                │
│     ┌──────────────┐  Parallel:                            │
│     │execute-batch │  ┌─────────┐ ┌─────────┐              │
│     │              │──│ task-1  │ │ task-2  │ ...          │
│     └──────────────┘  └────┬────┘ └────┬────┘              │
│                            │           │                    │
│                       ┌────▼────┐ ┌────▼────┐              │
│                       │ verify  │ │ verify  │              │
│                       └────┬────┘ └────┬────┘              │
│                            │           │                    │
│                       ┌────▼───────────▼────┐              │
│                       │    execute-merge    │ (sequential) │
│                       └─────────────────────┘              │
│   Output: Working code with tests                           │
└─────────────────────────────────────────────────────────────┘
```

### Brownfield Workflow

```
Existing Codebase
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /crd - Change Request Creation                              │
│   ┌───────────────────┐                                     │
│   │ Check PROJECT.md  │                                     │
│   └─────────┬─────────┘                                     │
│             │                                               │
│   ┌─────────▼─────────┐  (if missing)                      │
│   │  crd-investigate  │ → PROJECT.md                       │
│   └─────────┬─────────┘                                     │
│             │                                               │
│   ┌─────────▼─────────┐  (if stale)                        │
│   │crd-context-update │ → Updated PROJECT.md               │
│   └─────────┬─────────┘                                     │
│             │                                               │
│   ┌─────────▼─────────┐                                     │
│   │ Capture Change    │ → Change intent                    │
│   │ Requirements      │                                     │
│   └─────────┬─────────┘                                     │
│             │                                               │
│   ┌─────────▼─────────┐                                     │
│   │ crd-impact-       │ → Impact analysis                  │
│   │    analysis       │                                     │
│   └───────────────────┘                                     │
│   Output: docs/crd/{slug}/index.xml                        │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /breakdown (same as greenfield)                             │
│   - Reads CRD instead of PRD                               │
│   - Scoped by impact analysis                              │
│   Output: tasks/ directory with XML files                   │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ /execute (with context finalization)                        │
│   - Same parallel execution                                 │
│   - After completion:                                       │
│     ┌─────────────────────────┐                            │
│     │project-context-finalizer│ → Updated PROJECT.md       │
│     └─────────────────────────┘                            │
│   Output: Working code + updated context                    │
└─────────────────────────────────────────────────────────────┘
```

## State Management

State files track progress and enable resume:

| File | Purpose |
|------|---------|
| `analysis.json` | PRD/CRD analysis output |
| `layer_plan.json` | Layer organization |
| `manifest.json` | Task inventory |
| `execute-state.json` | Execution progress |

Example `execute-state.json`:

```json
{
  "status": "in_progress",
  "current_layer": 2,
  "tasks": {
    "L1-001-models": { "status": "completed" },
    "L2-001-auth-api": { "status": "in_progress" },
    "L2-002-user-api": { "status": "pending" }
  }
}
```

## Model Selection

Different components use different models:

| Component | Model | Reasoning |
|-----------|-------|-----------|
| `/prd`, `/crd` | Sonnet | Complex reasoning |
| breakdown sub-skills | Sonnet | Architecture planning |
| `breakdown-review-tasks` | Haiku | Fast validation |
| `execute-task` | Sonnet | Code implementation |
| `execute-verify` | Haiku | Fast verification |

This optimizes for both quality and speed.
