---
layout: default
title: Context Fork
parent: Introduction
nav_order: 2
---

# Context Fork

Context fork is the **key innovation** that enables PRD Breakdown Execute. It's a Claude Code feature that runs skills in completely isolated execution contexts.

## The Problem with Shared Context

In traditional AI conversations, context accumulates:

```
Turn 1: User asks about auth system
Turn 2: AI reads auth files
Turn 3: User asks about payment system
Turn 4: AI reads payment files
Turn 5: Context now includes BOTH systems
```

This creates problems:

- **Context pollution**: Unrelated information affects responses
- **Token bloat**: Context grows until it exceeds limits
- **Reasoning interference**: Past discussions influence new tasks
- **No parallelism**: Can't run multiple tasks simultaneously

## How Context Fork Works

Skills declare `context: fork` in their SKILL.md:

```yaml
---
name: execute-task
context: fork
model: sonnet
tools:
  - Read
  - Write
  - Bash
---
```

When invoked, this skill:

1. **Starts fresh** - No conversation history
2. **Gets only task input** - Just the XML task file
3. **Runs independently** - Other tasks don't see its work
4. **Returns results** - Output goes back to orchestrator

<img src="{{ '/assets/images/context-fork-diagram.svg' | relative_url }}" alt="Context Fork: Orchestrator spawns parallel task forks" style="width: 100%; max-width: 700px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

## Benefits

### True Parallelism

Without context fork, tasks must run sequentially because each task's context would influence the next. With forked contexts:

<img src="{{ '/assets/images/parallelism-comparison.svg' | relative_url }}" alt="Sequential vs Parallel execution comparison" style="width: 100%; max-width: 700px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

### No Cross-Contamination

Each task sees only its own files:

| What Task A Sees | What Task B Sees |
|-----------------|------------------|
| auth.py | payment.py |
| auth_test.py | payment_test.py |
| Task A XML | Task B XML |

No risk of Task A's auth code leaking into Task B's payment implementation.

### Independent Verification

Verification runs in its own fork:

<img src="{{ '/assets/images/verification-flow.svg' | relative_url }}" alt="Independent verification in separate fork" style="width: 100%; max-width: 500px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

The verifier doesn't know what the implementer was "trying" to do - it just checks if the code works.

### Error Containment

If Task A fails:

- Task B and C continue unaffected
- Task A retries with error feedback
- No cascade failures

<img src="{{ '/assets/images/error-containment.svg' | relative_url }}" alt="Error containment: failures don't cascade" style="width: 100%; max-width: 500px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

## Fork Hierarchy

The execute pipeline uses nested forks:

<img src="{{ '/assets/images/fork-hierarchy.svg' | relative_url }}" alt="Fork hierarchy: nested execution pipeline" style="width: 100%; max-width: 600px; margin: 1.5rem auto; display: block; border-radius: 12px;" />

Each level manages its own scope:
- **/execute**: Overall orchestration
- **execute-layer**: Layer-by-layer coordination
- **execute-batch**: Parallel task batching
- **execute-task**: Individual implementation
- **execute-verify**: Independent verification
- **execute-merge**: Sequential merge (must be atomic)

## Declaring Fork in Skills

To create a forked skill:

```yaml
---
name: my-isolated-skill
context: fork
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
---

# My Isolated Skill

This skill runs in complete isolation.
It cannot see the parent conversation.
It gets only the input provided via the Task tool.
```

## Best Practices

1. **Fork for parallelism** - Use fork when tasks can run simultaneously
2. **Fork for isolation** - Use fork when context pollution is a risk
3. **Don't fork for orchestration** - Orchestrators need to see results
4. **Limit tool access** - Forked skills should only get necessary tools
5. **Design for self-containment** - Forked skills can't ask the orchestrator questions
