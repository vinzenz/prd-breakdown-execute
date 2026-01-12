---
layout: default
title: Quick Start
nav_order: 2
has_children: true
permalink: /docs/quickstart/
---

# Quick Start

Get up and running with PRD Breakdown Execute in 5 minutes.

## Prerequisites

- **Claude Code CLI** installed and authenticated
- **Git** for worktree-based parallel execution
- A project directory to work in

## Installation

Clone the repository and add the skills to your Claude Code:

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/prd-breakdown-execute.git

# Copy skills to your Claude Code directory
cp -r prd-breakdown-execute/.claude/* ~/.claude/
```

Or add as a submodule to your project:

```bash
# Add as submodule
git submodule add https://github.com/YOUR_USERNAME/prd-breakdown-execute.git .prd-skills

# Symlink the .claude directory
ln -s .prd-skills/.claude .claude
```

## Your First Project (Greenfield)

### Step 1: Create a PRD

```bash
cd your-project
claude /prd
```

The interactive wizard guides you through 8 phases:

1. **Idea capture** - Describe your project
2. **Tech stack** - Choose your framework
3. **Features** - Define what to build
4. **Dependencies** - External services needed
5. **Options** - Configuration preferences
6. **Review** - Validate completeness
7. **Feature details** - Deep dive each feature
8. **Output** - Generate PRD files

Output: `docs/prd/{slug}/` directory with `index.md` and `features/`

### Step 2: Break Down into Tasks

```bash
claude /breakdown docs/prd/my-project/
```

This:
1. Analyzes your PRD
2. Plans dependency layers
3. Generates self-contained task XML files
4. Reviews for quality

Output: `docs/prd/my-project/tasks/` with layer directories and `.xml` files

### Step 3: Execute Tasks

```bash
claude /execute docs/prd/my-project/tasks/
```

This:
1. Creates git worktrees for parallel execution
2. Runs tasks layer by layer
3. Implements with TDD (tests first)
4. Verifies independently
5. Merges to main branch

Output: Working code with tests in your project

## Your First Change (Brownfield)

For existing codebases, use the CRD workflow:

### Step 1: Create a CRD

```bash
cd existing-project
claude /crd
```

This:
1. Analyzes your codebase (creates `PROJECT.md` if needed)
2. Captures your change request
3. Runs impact analysis
4. Generates a CRD document

Output: `docs/crd/{slug}/index.xml`

### Step 2: Break Down and Execute

```bash
# Break down the CRD
claude /breakdown docs/crd/my-change/

# Execute the tasks
claude /execute docs/crd/my-change/tasks/
```

The workflow is the same as greenfield, but:
- Tasks are scoped by impact analysis
- `PROJECT.md` is updated after completion

## Quick Reference

| Command | Purpose | Output |
|---------|---------|--------|
| `/prd` | Create PRD interactively | `docs/prd/{slug}/` |
| `/crd` | Create change request | `docs/crd/{slug}/` |
| `/crd-context` | Manage PROJECT.md | `PROJECT.md` |
| `/breakdown <path>` | Generate tasks | `{path}/tasks/` |
| `/execute <path>` | Run tasks | Working code |

## Next Steps

- [Create Your First PRD]({{ '/docs/quickstart/first-prd/' | relative_url }}) - Detailed walkthrough
- [Understanding Breakdown]({{ '/docs/quickstart/first-breakdown/' | relative_url }}) - How tasks are generated
- [Running Execute]({{ '/docs/quickstart/first-execute/' | relative_url }}) - Parallel execution in depth
