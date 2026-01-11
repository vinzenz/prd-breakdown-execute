# Agent Specialization

Matching model capabilities to task requirements.

---

## The Principle

**Use the right model for each job.**

| Task Type | Model | Reasoning |
|-----------|-------|-----------|
| Architecture | Sonnet | Complex reasoning |
| Implementation | Sonnet | Code generation |
| Verification | Haiku | Fast, focused |
| Review | Haiku | Quick validation |

---

## Model Characteristics

### Sonnet (Claude 3.5/4)

**Strengths:**
- Complex reasoning
- Code generation
- Architecture decisions
- Long-form planning
- Nuanced understanding

**Use for:**
- Orchestration (execute, execute-layer)
- Code implementation (execute-task)
- Task generation (breakdown-generate-tasks)
- Analysis (breakdown-analyze-prd)
- Merging (execute-merge)

**Trade-off:** More expensive, slower

### Haiku (Claude 3)

**Strengths:**
- Speed
- Focused tasks
- Pattern matching
- Simple decisions
- Cost efficiency

**Use for:**
- Verification (execute-verify)
- Review (breakdown-review-tasks)
- Quality checks

**Trade-off:** Less capable for complex tasks

---

## Skill Assignments

```
/prd (command)
    │ Model: Sonnet
    │ Why: Interactive, needs nuanced understanding
    │
/breakdown (skill)
    │ Model: Sonnet
    │ Why: Architecture decisions
    │
    ├── breakdown-analyze-prd
    │       Model: Sonnet
    │       Why: Feature extraction, inference
    │
    ├── breakdown-plan-layers
    │       Model: Sonnet
    │       Why: Dependency analysis
    │
    ├── breakdown-generate-tasks
    │       Model: Sonnet
    │       Why: Detailed specification
    │
    └── breakdown-review-tasks
            Model: Haiku
            Why: Checklist validation

/execute (skill)
    │ Model: Sonnet
    │ Why: Orchestration logic
    │
    ├── execute-layer
    │       Model: Sonnet
    │       Why: Batch coordination
    │
    ├── execute-batch
    │       Model: Sonnet
    │       Why: Parallel task management
    │
    ├── execute-task
    │       Model: Sonnet
    │       Why: Code implementation
    │
    ├── execute-verify
    │       Model: Haiku
    │       Why: Run commands, check output
    │
    └── execute-merge
            Model: Sonnet
            Why: Git operations, conflict handling
```

---

## Why Haiku for Verification?

### 1. Speed

Verification runs after every implementation:

```
Task implementation: ~30-60 seconds (Sonnet)
Verification: ~5-10 seconds (Haiku)
```

Using Sonnet for verification would double execution time.

### 2. Focused Task

Verification is simple:

```
1. Run pytest command
2. Check exit code
3. Look for expected patterns
4. Report pass/fail
```

Doesn't need complex reasoning.

### 3. Cost Efficiency

```
Sonnet cost: $$
Haiku cost:  $

With 15 tasks, each verified:
  Sonnet verification: $$$
  Haiku verification:  $
```

### 4. Independence

Using a different model adds objectivity:

```
Sonnet: "I implemented this correctly"
Haiku: "Let me check independently"
```

Different perspective, less bias.

---

## Why Haiku for Review?

### Review is Checklist-Based

```
[ ] Meta section complete?
[ ] No placeholders?
[ ] All imports included?
[ ] Types annotated?
[ ] Tests have concrete values?
```

Pattern matching, not creative reasoning.

### Speed Matters

Review happens for every batch:

```
Generate 5 tasks → Review
Generate 5 tasks → Review
Generate 5 tasks → Review
```

Fast review = faster iteration.

---

## Why Sonnet for Implementation?

### Code Generation

Writing code requires:
- Understanding context
- Making design decisions
- Handling edge cases
- Following conventions

```python
# Sonnet can generate complex code:
@router.post("/", response_model=TaskResponse, status_code=201)
async def create_task(
    request: TaskCreateRequest,
    repository: TaskRepository = Depends(get_repository)
) -> TaskResponse:
    task = Task(
        id=uuid4(),
        title=request.title,
        description=request.description,
        status=TaskStatus.PENDING,
        created_at=datetime.utcnow()
    )
    created = await repository.create(task)
    return TaskResponse.from_orm(created)
```

### Error Recovery

When verification fails, Sonnet can:
- Understand the feedback
- Identify the root cause
- Generate a fix
- Apply it correctly

---

## Cost vs Quality Trade-off

```
Task                    Model    Quality  Speed  Cost
────────────────────────────────────────────────────
PRD creation            Sonnet   High     Slow   $$
Task generation         Sonnet   High     Slow   $$
Task review             Haiku    Good     Fast   $
Code implementation     Sonnet   High     Slow   $$
Verification            Haiku    Good     Fast   $
Merge operations        Sonnet   High     Slow   $$
```

### Optimization Strategy

1. **Use Sonnet where quality matters most**
   - Implementation (code correctness)
   - Analysis (feature extraction)
   - Orchestration (state management)

2. **Use Haiku where speed matters most**
   - Verification (frequent operation)
   - Review (iterative process)

3. **Never compromise on output quality**
   - Final code is always Sonnet-generated
   - Haiku only validates, never generates

---

## Declaring Model in Skills

```yaml
# In SKILL.md
---
name: execute-verify
context: fork
model: haiku       # ← Model specification
allowed-tools:
  - Bash
  - Read
---
```

Each skill explicitly declares its model requirement.

---

## Agent Definitions

The `agents/` directory contains specialized agent definitions:

```
.claude/agents/
├── task-generator.md      # Creates task XML (Sonnet)
├── task-implementer.md    # Implements tasks (Haiku target)
├── task-reviewer.md       # Reviews tasks (Haiku)
└── verification-runner.md # Runs verification (Haiku)
```

These are used by the Task tool for specific purposes.

---

## Best Practices

### 1. Match Capability to Need

Don't use Sonnet for simple validation.
Don't use Haiku for complex code generation.

### 2. Consider Frequency

High-frequency operations → optimize for speed
Low-frequency operations → optimize for quality

### 3. Maintain Independence

Different models for implementation and verification
→ Reduces confirmation bias

### 4. Monitor Performance

Track which tasks fail and why
→ Adjust model assignments if needed

---

## Next Steps

- [Context Fork](../introduction/context-fork.md) - How models run in isolation
- [Execute Hierarchy](../skills/execute/hierarchy.md) - Model distribution in practice
- [Self-Contained Tasks](self-contained-tasks.md) - Why small context works
