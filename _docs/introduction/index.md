---
layout: default
title: Introduction
nav_order: 1
has_children: true
permalink: /docs/introduction/
---

# Introduction

PRD Breakdown Execute is a **Claude Code skills framework** demonstrating autonomous software development. It transforms product requirements documents (PRDs) or change request documents (CRDs) into working, tested code through an intelligent pipeline.

## The Problem

Traditional AI-assisted coding has limitations:

- **Context overload**: Large codebases exceed context windows
- **Sequential bottleneck**: Tasks execute one at a time
- **Lost state**: Conversations don't persist across sessions
- **Manual coordination**: Humans must orchestrate AI tasks

## The Solution

PRD Breakdown Execute solves these through:

| Problem | Solution |
|---------|----------|
| Context overload | **Context fork** - isolated execution contexts |
| Sequential bottleneck | **Parallel execution** in git worktrees |
| Lost state | **State files** (JSON) track progress |
| Manual coordination | **Skill orchestration** automates workflow |

## How It Works

<img src="{{ '/assets/images/how-it-works.svg' | relative_url }}" alt="PRD Breakdown Execute: Three-phase pipeline" style="width: 100%; max-width: 800px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

## Key Concepts

### Context Fork

Skills declare `context: fork` to run in isolated contexts. This enables:

- **Parallel execution** without interference
- **Fresh context** for each task
- **Independent verification**

[Learn more about context fork →]({{ '/docs/introduction/context-fork/' | relative_url }})

### Self-Contained Tasks

Each task includes everything needed for implementation:

- PRD context excerpt
- Interface contracts from dependencies
- Test requirements (TDD)
- Verification commands

[Learn more about self-contained tasks →]({{ '/docs/concepts/self-contained-tasks/' | relative_url }})

### 5-Layer Architecture

Tasks are organized into dependency layers:

```
L4: Integration  ← E2E flows, final wiring
L3: Frontend     ← UI components, state
L2: Backend      ← APIs, services, logic
L1: Foundation   ← Models, migrations, config
L0: Setup        ← Project initialization
```

[Learn more about layers →]({{ '/docs/skills/breakdown/layers/' | relative_url }})

## Next Steps

- [Quick Start]({{ '/docs/quickstart/' | relative_url }}) - Get running in 5 minutes
- [Architecture]({{ '/docs/introduction/architecture/' | relative_url }}) - Deep dive into system design
- [Examples]({{ '/docs/examples/' | relative_url }}) - See complete walkthroughs
