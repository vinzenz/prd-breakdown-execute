# CLAUDE.md

Instructions for Claude when working in this repository.

---

## Project Overview

This repository contains Claude Code skills demonstrating an autonomous software development pipeline:

1. **`/prd`** - Interactive PRD creation command
2. **`/breakdown`** - PRD to task conversion skill
3. **`/execute`** - Parallel task execution skill

The key innovation demonstrated is **context fork** - skills that run in isolated contexts.

---

## Key Directories

```
.claude/
├── commands/         # User-invocable commands (/prd)
├── skills/           # Skills with SKILL.md definitions
│   ├── breakdown/    # Main breakdown skill
│   ├── execute/      # Main execute skill
│   └── ...           # Sub-skills
└── agents/           # Agent definitions for Task tool

docs/
├── IMPLEMENTATION_GUIDE.md  # Guide for creating documentation
├── introduction/            # Conceptual documentation
├── quickstart/              # Getting started guides
├── skills/                  # Skill reference documentation
├── concepts/                # Core concept explanations
├── examples/                # Full walkthroughs
└── reference/               # Technical reference
```

---

## Skill Architecture

### Context Modes

Skills declare their context mode in SKILL.md frontmatter:

```yaml
---
name: execute-task
context: fork        # ← Runs in isolated context
model: sonnet
---
```

- **`context: fork`** - Isolated context, no parent visibility
- **Default** - Shares context with parent

### Skill Hierarchy

```
/execute (fork)
    └─► execute-layer (fork)
            └─► execute-batch (fork)
                    ├─► execute-task (fork)
                    │       └─► execute-verify (fork)
                    └─► execute-merge (fork)
```

### Reference Files

Skills can have reference files in `references/` subdirectory:

```
.claude/skills/breakdown/
├── SKILL.md
└── references/
    ├── layer-definitions.md
    └── task-format-spec.md
```

---

## Conventions

### Naming

- Skill files: `SKILL.md` (exactly)
- Commands: lowercase with hyphens (e.g., `prd.md`)
- Agents: lowercase with hyphens (e.g., `task-generator.md`)
- Layers: `{n}-{name}` (e.g., `2-backend`)
- Tasks: `L{layer}-{seq}-{slug}.xml` (e.g., `L2-003-project-crud.xml`)

### Task XML Format

Tasks are XML with these sections:
- `<meta>` - ID, name, layer, priority
- `<context>` - PRD excerpt, tech stack
- `<dependencies>` - Interface contracts from previous tasks
- `<objective>` - What to achieve
- `<requirements>` - Detailed specifications
- `<test-requirements>` - TDD test cases
- `<files-to-create>` - File paths (max 3)
- `<verification>` - Runnable commands
- `<exports>` - Interface contracts for downstream

### State Files

- `analysis.json` - PRD analysis output
- `layer_plan.json` - Layer organization
- `manifest.json` - Task inventory
- `execute-state.json` - Execution state

---

## Testing Documentation

When editing documentation:

1. **Verify internal links**
   ```bash
   # Check that linked files exist
   grep -r '\[.*\](.*\.md)' docs/ | while read line; do
     # Extract and verify paths
   done
   ```

2. **Validate code examples**
   - Commands match actual skill definitions
   - XML examples match task-format-spec.md

3. **Check ASCII diagrams**
   - Render correctly in terminal
   - Alignment is preserved

4. **Consistent terminology**
   - "context fork" (not "context forking")
   - "skill" (not "plugin")
   - "task" (not "job")

---

## Common Tasks

### Adding Documentation

1. Follow structure in `docs/IMPLEMENTATION_GUIDE.md`
2. Use ASCII diagrams for architecture
3. Include practical examples
4. Link to related concepts

### Updating Skill Documentation

When a skill changes:
1. Update corresponding doc in `docs/skills/`
2. Check if reference files need updating
3. Verify examples still work

### Adding Examples

1. Create in `docs/examples/`
2. Include complete workflow (PRD → Breakdown → Execute)
3. Show actual file contents, not placeholders
4. Document expected outcomes

---

## Important Notes

- This repo demonstrates Claude Code features
- The context fork pattern is the key innovation
- All skills use explicit tool allowlists
- State management enables resume capability
- TDD is mandatory in execute-task
