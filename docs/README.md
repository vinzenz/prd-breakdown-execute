# Documentation

Welcome to the PRD Breakdown Execute documentation.

---

## Quick Navigation

### New Here?
- [Introduction](introduction/README.md) - What this project does
- [Quick Start](quickstart/README.md) - Get running in 5 minutes

### Understanding the Innovation
- [Context Fork](introduction/context-fork.md) - The key feature that makes this work

### Reference
- [Skills Overview](skills/README.md) - All skills at a glance
- [Commands Reference](reference/commands.md) - All commands and arguments

### Learning by Example
- [Todo App Walkthrough](examples/todo-app.md) - Complete end-to-end example

---

## Documentation Map

```
docs/
│
├── introduction/
│   ├── README.md           # What is this project?
│   ├── context-fork.md     # The key innovation
│   └── architecture.md     # System overview
│
├── quickstart/
│   ├── README.md           # 5-minute start
│   ├── first-prd.md        # Create your first PRD
│   ├── first-breakdown.md  # Generate tasks
│   └── first-execute.md    # Run execution
│
├── skills/
│   ├── README.md           # Skill overview
│   ├── prd.md              # /prd reference
│   │
│   ├── breakdown/
│   │   ├── README.md       # Breakdown overview
│   │   ├── layers.md       # 5-layer architecture
│   │   ├── task-format.md  # Task XML specification
│   │   └── sub-skills.md   # Sub-skill documentation
│   │
│   └── execute/
│       ├── README.md       # Execute overview
│       ├── hierarchy.md    # 4-level hierarchy
│       ├── worktrees.md    # Git worktree isolation
│       ├── tdd.md          # TDD workflow
│       ├── state.md        # State management
│       └── sub-skills.md   # Sub-skill documentation
│
├── concepts/
│   ├── README.md                 # Concept index
│   ├── self-contained-tasks.md   # Task philosophy
│   ├── interface-contracts.md    # Type boundaries
│   └── agent-specialization.md   # Model selection
│
├── examples/
│   ├── README.md           # Example index
│   └── todo-app.md         # Full walkthrough
│
├── reference/
│   ├── README.md           # Reference index
│   ├── commands.md         # Command reference
│   └── file-formats.md     # Schema reference
│
└── troubleshooting.md      # Common issues
```

---

## Start Here

### I want to...

| Goal | Start Here |
|------|------------|
| Understand what this does | [Introduction](introduction/README.md) |
| Get running quickly | [Quick Start](quickstart/README.md) |
| Learn about context fork | [Context Fork](introduction/context-fork.md) |
| See a full example | [Todo App](examples/todo-app.md) |
| Reference a specific skill | [Skills](skills/README.md) |
| Troubleshoot an issue | [Troubleshooting](troubleshooting.md) |

---

## Key Concepts

| Concept | What It Means |
|---------|---------------|
| **Context Fork** | Skills run in isolated contexts, enabling true parallelism |
| **5 Layers** | Setup → Foundation → Backend → Frontend → Integration |
| **Self-Contained Tasks** | Each task has all context needed for autonomous execution |
| **Interface Contracts** | Type signatures shared between tasks, not implementations |
| **TDD Verification** | Tests first, then implementation, then independent verification |

---

## The Workflow

```
    /prd                    /breakdown                  /execute
      │                         │                          │
      ▼                         ▼                          ▼
┌───────────┐            ┌───────────┐             ┌───────────┐
│ Interactive│            │  Analyze  │             │Orchestrate│
│  8 phases  │ ────────>  │   Plan    │ ────────>   │  Parallel │
│            │            │  Generate │             │    TDD    │
└───────────┘            │  Review   │             └───────────┘
      │                   └───────────┘                   │
      ▼                         │                         ▼
 docs/prd/                      ▼                    Working
  index.md                 docs/tasks/                 Code
                            *.xml
```

---

## External Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [ARCHITECTURE.md](../ARCHITECTURE.md) - Technical deep dive
- [CLAUDE.md](../CLAUDE.md) - Instructions for Claude
