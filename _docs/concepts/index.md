---
layout: default
title: Concepts
nav_order: 4
has_children: true
permalink: /docs/concepts/
---

# Core Concepts

Understanding these concepts will help you get the most out of PRD Breakdown Execute.

## The Three Pillars

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRD Breakdown Execute                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │                 │  │                 │  │                 │  │
│  │  Context Fork   │  │  Self-Contained │  │  Interface      │  │
│  │                 │  │     Tasks       │  │  Contracts      │  │
│  │  Isolated       │  │  Everything     │  │  Type-safe      │  │
│  │  execution      │  │  in one file    │  │  boundaries     │  │
│  │                 │  │                 │  │                 │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Context Fork

Skills run in isolated contexts, enabling parallel execution without interference.

- No shared conversation history
- Each task sees only its own files
- Verification is unbiased

[Learn more →]({{ '/docs/introduction/context-fork/' | relative_url }})

### Self-Contained Tasks

Each task XML includes everything needed for implementation:

- PRD excerpt (relevant context only)
- Interface contracts from dependencies
- Test requirements (TDD)
- Verification commands

[Learn more →]({{ '/docs/concepts/self-contained-tasks/' | relative_url }})

### Interface Contracts

Type signatures pass between tasks as contracts:

```xml
<dependency task="L1-001-user-model">
  <interface-contract>
    class User(Base):
        id: UUID
        email: str
  </interface-contract>
</dependency>
```

[Learn more →]({{ '/docs/concepts/interface-contracts/' | relative_url }})

## Additional Concepts

### Agent Specialization

Different models for different jobs:
- **Sonnet**: Complex reasoning, implementation
- **Haiku**: Fast validation, verification

[Learn more →]({{ '/docs/concepts/agent-specialization/' | relative_url }})

### 5-Layer Architecture

Tasks organized by dependencies:

```
L4: Integration  ← E2E flows
L3: Frontend     ← UI components
L2: Backend      ← APIs, services
L1: Foundation   ← Models, config
L0: Setup        ← Scaffolding
```

[Learn more →]({{ '/docs/skills/breakdown/layers/' | relative_url }})

### State Management

JSON files track progress for resume capability:

- `analysis.json` - PRD analysis
- `layer_plan.json` - Layer organization
- `manifest.json` - Task inventory
- `execute-state.json` - Execution progress

### Git Worktrees

Parallel tasks run in isolated git worktrees:

```
.worktrees/
├── task-L1-001/  # Isolated copy
├── task-L1-002/  # Isolated copy
└── task-L1-003/  # Isolated copy
```

### TDD Workflow

All tasks follow test-driven development:

1. Write failing tests
2. Run tests (expect red)
3. Write implementation
4. Run tests (expect green)
5. Commit

### PROJECT.md (Brownfield)

Machine-readable codebase context:

```markdown
# Project Name

## Architecture
[Human-readable description]

<project-context>
  <features>...</features>
  <api-registry>...</api-registry>
  <schema-registry>...</schema-registry>
</project-context>
```
