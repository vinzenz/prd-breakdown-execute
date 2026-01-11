# Core Concepts

Key ideas that make the PRD Breakdown Execute workflow work.

---

## Concept Overview

| Concept | Description |
|---------|-------------|
| [Context Fork](../introduction/context-fork.md) | Isolated execution contexts for parallelism |
| [Self-Contained Tasks](self-contained-tasks.md) | Everything needed in one file |
| [Interface Contracts](interface-contracts.md) | Type signatures as boundaries |
| [Agent Specialization](agent-specialization.md) | Right model for each job |

---

## How They Work Together

```
┌─────────────────────────────────────────────────────────────────┐
│                      Context Fork                               │
│                                                                 │
│  Enables parallel execution without interference                │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                Self-Contained Tasks                         ││
│  │                                                             ││
│  │  Each task has all context needed - works in any fork       ││
│  │                                                             ││
│  │  ┌─────────────────────────────────────────────────────────┐││
│  │  │              Interface Contracts                        │││
│  │  │                                                         │││
│  │  │  Tasks communicate through type signatures              │││
│  │  │  Not implementation details                             │││
│  │  └─────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              Agent Specialization                           ││
│  │                                                             ││
│  │  Sonnet: Architecture, implementation                       ││
│  │  Haiku: Verification, quick checks                          ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

### Context Fork

- Skills declare `context: fork` in SKILL.md
- Each fork is completely isolated
- No context pollution between tasks
- Enables true parallel execution

### Self-Contained Tasks

- All context inline in task XML
- No external file lookups
- Designed for ~50k token models
- Implementing agent cannot ask questions

### Interface Contracts

- Type signatures, not implementations
- Dependencies section: what task receives
- Exports section: what task provides
- Enables independent development

### Agent Specialization

- Sonnet: Complex reasoning, code generation
- Haiku: Fast validation, simple checks
- Match model capability to task complexity
- Optimize for cost and performance

---

## Learn More

- [Context Fork](../introduction/context-fork.md) - The key innovation
- [Self-Contained Tasks](self-contained-tasks.md) - Task philosophy
- [Interface Contracts](interface-contracts.md) - Communication between tasks
- [Agent Specialization](agent-specialization.md) - Model selection
