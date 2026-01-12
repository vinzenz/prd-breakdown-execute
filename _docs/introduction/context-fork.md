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

<div class="diagram">
<pre>
┌─────────────────────────────────────────────────────────┐
│  Orchestrator (main context)                            │
│                                                         │
│  "Execute these 3 tasks in parallel"                    │
│          │              │              │                │
│          ▼              ▼              ▼                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │ Task Fork 1 │ │ Task Fork 2 │ │ Task Fork 3 │       │
│  │             │ │             │ │             │       │
│  │ • Empty ctx │ │ • Empty ctx │ │ • Empty ctx │       │
│  │ • Task XML  │ │ • Task XML  │ │ • Task XML  │       │
│  │ • Tools     │ │ • Tools     │ │ • Tools     │       │
│  └─────────────┘ └─────────────┘ └─────────────┘       │
│          │              │              │                │
│          └──────────────┴──────────────┘                │
│                         │                               │
│                         ▼                               │
│              Results merged back                        │
└─────────────────────────────────────────────────────────┘
</pre>
</div>

## Benefits

### True Parallelism

Without context fork, tasks must run sequentially because each task's context would influence the next. With forked contexts:

<div class="diagram-blueprint">
<pre>
Sequential (no fork):        Parallel (with fork):

Task A ████████████          Task A ████████████
         Task B ████████████   Task B ████████████
                   Task C      Task C ████████████

Time: 24 units               Time: 8 units
</pre>
</div>

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

<div class="diagram">
<pre>
┌─────────────────────────────────────────────┐
│ execute-task (fork)                         │
│   • Writes implementation                   │
│   • Creates tests                           │
│       │                                     │
│       ▼                                     │
│ ┌───────────────────────────────────┐       │
│ │ execute-verify (fork)             │       │
│ │   • Fresh context                 │       │
│ │   • Runs verification commands    │       │
│ │   • No implementation bias        │       │
│ └───────────────────────────────────┘       │
└─────────────────────────────────────────────┘
</pre>
</div>

The verifier doesn't know what the implementer was "trying" to do - it just checks if the code works.

### Error Containment

If Task A fails:

- Task B and C continue unaffected
- Task A retries with error feedback
- No cascade failures

<div class="diagram">
<pre>
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Task A  │  │ Task B  │  │ Task C  │
│  FAIL   │  │  PASS   │  │  PASS   │
│    ↓    │  │    ✓    │  │    ✓    │
│  Retry  │  │         │  │         │
│  PASS   │  │         │  │         │
└─────────┘  └─────────┘  └─────────┘
</pre>
</div>

## Fork Hierarchy

The execute pipeline uses nested forks:

<div class="diagram-flow">
<pre>
/execute (fork)
    └─► execute-layer (fork)
            └─► execute-batch (fork)
                    ├─► execute-task (fork)
                    │       └─► execute-verify (fork)
                    └─► execute-merge (fork)
</pre>
</div>

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
