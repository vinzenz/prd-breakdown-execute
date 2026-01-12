---
layout: default
title: Examples
nav_order: 5
has_children: true
permalink: /docs/examples/
---

# Examples

Complete walkthroughs demonstrating PRD Breakdown Execute in action.

## Available Examples

### Todo App (Greenfield)

A complete task management API built from scratch using the greenfield workflow.

**Demonstrates:**
- Interactive PRD creation with `/prd`
- Task breakdown with layer planning
- Parallel execution with TDD
- Full API implementation

[View Todo App Example →]({{ '/docs/examples/todo-app/' | relative_url }})

## Example Overview

Each example follows the complete workflow:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Example Flow                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Define Requirements                                    │
│  ├── Run /prd or /crd                                          │
│  └── Output: docs/prd/{name}/ or docs/crd/{name}/              │
│                                                                  │
│  Step 2: Break Down                                             │
│  ├── Run /breakdown                                             │
│  ├── Analyze features                                           │
│  ├── Plan layers                                                │
│  ├── Generate tasks                                             │
│  └── Output: tasks/ directory with XML files                    │
│                                                                  │
│  Step 3: Execute                                                │
│  ├── Run /execute                                               │
│  ├── Create worktrees                                           │
│  ├── Implement with TDD                                         │
│  ├── Verify independently                                       │
│  ├── Merge to main                                              │
│  └── Output: Working code with tests                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## What You'll Learn

| Example | Key Lessons |
|---------|-------------|
| Todo App | Basic workflow, layer architecture, TDD |

## Running Examples

Examples can be run yourself:

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/prd-breakdown-execute.git
cd prd-breakdown-execute

# Copy skills to Claude Code
cp -r .claude/* ~/.claude/

# Start fresh project
mkdir my-example
cd my-example
git init

# Follow the example steps
claude /prd
```
