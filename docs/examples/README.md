# Examples

Complete walkthroughs demonstrating the PRD Breakdown Execute workflow.

---

## Available Examples

| Example | Description | Complexity |
|---------|-------------|------------|
| [Todo App](todo-app.md) | Full-stack task management API | Medium |

---

## What Examples Show

Each example includes:

1. **Initial Idea** - The starting point
2. **PRD Creation** - Key phases and decisions
3. **Generated PRD** - Actual output files
4. **Breakdown Output** - Analysis and task files
5. **Execution Log** - What happens during execution
6. **Final Result** - Generated code structure

---

## Running Examples

### Prerequisites

1. Claude Code CLI installed
2. Git configured
3. Development dependencies for chosen stack

### Steps

```bash
# 1. Create a new project directory
mkdir my-project && cd my-project

# 2. Copy skills
cp -r /path/to/prd-breakdown-execute/.claude .

# 3. Create PRD
claude /prd

# 4. Break it down
claude /breakdown docs/prd/my-project/index.md

# 5. Execute
claude /execute docs/tasks/my-project
```

---

## Learning Path

1. **Start with Todo App** - Simple but complete example
2. **Read the PRD phases** - Understand how requirements are captured
3. **Examine task XML** - See how tasks are structured
4. **Follow execution** - Understand parallel processing

---

## Creating Your Own

After understanding examples:

1. Start with a clear problem statement
2. Be specific in feature definitions
3. Use concrete acceptance criteria
4. Let breakdown infer what it can
5. Review generated tasks before executing

---

## Next Steps

- [Todo App Example](todo-app.md) - Complete walkthrough
- [Quick Start](../quickstart/README.md) - Get running fast
- [PRD Reference](../skills/prd.md) - All PRD options
