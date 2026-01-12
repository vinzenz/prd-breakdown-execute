---
layout: default
title: Home
nav_order: 1
description: "Autonomous Software Development with Claude Code Skills"
permalink: /
---

<div class="hero">
  <h1>PRD Breakdown Execute</h1>
  <p class="tagline">Transform ideas into working code through AI-orchestrated parallel development</p>
  <div class="cta-buttons">
    <a href="{{ '/docs/quickstart/' | relative_url }}" class="primary">Get Started</a>
    <a href="{{ '/docs/introduction/' | relative_url }}" class="secondary">Learn More</a>
  </div>
</div>

## What is PRD Breakdown Execute?

PRD Breakdown Execute is a **Claude Code skills framework** that demonstrates autonomous software development. It transforms product requirements into working, tested code through an intelligent three-phase pipeline.

<div class="workflow-diagram">
  <span class="step">/prd</span>
  <span class="arrow">→</span>
  <span class="step">/breakdown</span>
  <span class="arrow">→</span>
  <span class="step">/execute</span>
  <span class="arrow">→</span>
  <span class="step">Working Code</span>
</div>

---

## Key Innovation: Context Fork

The framework's power comes from **context fork** - a Claude Code feature that runs skills in completely isolated execution contexts:

```
┌─────────────────────────────────────────────────────────────────┐
│  Main Session                                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  /execute                                                    ││
│  │  ┌───────────────────┐  ┌───────────────────┐               ││
│  │  │ Task A (fork)     │  │ Task B (fork)     │  ← Parallel   ││
│  │  │ - Fresh context   │  │ - Fresh context   │               ││
│  │  │ - Own worktree    │  │ - Own worktree    │               ││
│  │  │ - No interference │  │ - No interference │               ││
│  │  └───────────────────┘  └───────────────────┘               ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

This enables **true parallel execution** where multiple tasks run simultaneously without context pollution.

---

## Two Workflows

<div class="features">
  <div class="feature-card">
    <div class="icon">🌱</div>
    <h3><span class="badge greenfield">Greenfield</span> New Projects</h3>
    <p>Start from an idea. The interactive <code>/prd</code> command guides you through 8 phases to create comprehensive product requirements, then breaks them down and executes them.</p>
  </div>
  <div class="feature-card">
    <div class="icon">🏗️</div>
    <h3><span class="badge brownfield">Brownfield</span> Existing Projects</h3>
    <p>Add features to existing codebases. The <code>/crd</code> command analyzes your code, understands context, and creates change requests with full impact analysis.</p>
  </div>
</div>

---

## Features at a Glance

<div class="features">
  <div class="feature-card">
    <div class="icon">⚡</div>
    <h3>Parallel Execution</h3>
    <p>Multiple tasks run simultaneously in isolated git worktrees, maximizing development speed.</p>
  </div>
  <div class="feature-card">
    <div class="icon">🧪</div>
    <h3>TDD by Default</h3>
    <p>Tests are written first, implementation follows, and verification runs independently.</p>
  </div>
  <div class="feature-card">
    <div class="icon">🔒</div>
    <h3>Context Isolation</h3>
    <p>Each task gets fresh, focused context. No cross-contamination between parallel tasks.</p>
  </div>
  <div class="feature-card">
    <div class="icon">🔄</div>
    <h3>Self-Healing</h3>
    <p>Failed tasks retry up to 5 times with actionable feedback from verification.</p>
  </div>
  <div class="feature-card">
    <div class="icon">💾</div>
    <h3>Resume Capability</h3>
    <p>Stop and continue without losing progress. State-based tracking ensures nothing is lost.</p>
  </div>
  <div class="feature-card">
    <div class="icon">🤖</div>
    <h3>Model Specialization</h3>
    <p>Sonnet for complex reasoning, Haiku for fast verification. Right model for each job.</p>
  </div>
</div>

---

## 5-Layer Architecture

Tasks are organized into sequential layers to manage dependencies:

<div class="layer-stack">
  <div class="layer layer-4">
    <span class="layer-num">L4</span>
    <span class="layer-name">Integration</span>
    <span class="layer-desc">E2E flows, final wiring</span>
  </div>
  <div class="layer layer-3">
    <span class="layer-num">L3</span>
    <span class="layer-name">Frontend</span>
    <span class="layer-desc">UI components, state management</span>
  </div>
  <div class="layer layer-2">
    <span class="layer-num">L2</span>
    <span class="layer-name">Backend</span>
    <span class="layer-desc">APIs, services, business logic</span>
  </div>
  <div class="layer layer-1">
    <span class="layer-num">L1</span>
    <span class="layer-name">Foundation</span>
    <span class="layer-desc">Models, migrations, configuration</span>
  </div>
  <div class="layer layer-0">
    <span class="layer-num">L0</span>
    <span class="layer-name">Setup</span>
    <span class="layer-desc">Project initialization</span>
  </div>
</div>

Each layer depends only on the layer below. Within a layer, tasks run in **parallel**.

---

## Quick Example

```bash
# 1. Create a PRD interactively
claude /prd

# 2. Break down into executable tasks
claude /breakdown docs/prd/my-project/

# 3. Execute all tasks in parallel
claude /execute docs/prd/my-project/tasks/
```

That's it. From idea to working code with tests.

---

## Ready to Start?

<div class="cta-buttons" style="justify-content: flex-start; margin-top: 2rem;">
  <a href="{{ '/docs/quickstart/' | relative_url }}" class="primary">Quick Start Guide →</a>
</div>
