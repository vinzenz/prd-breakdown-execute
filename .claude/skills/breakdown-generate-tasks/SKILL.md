---
name: breakdown-generate-tasks
description: Generate implementation task files for a specific layer. Called by /breakdown skill during Phase 4.
context: fork
agent: task-generator
allowed-tools: Read Write Glob Grep
model: claude-sonnet-4-5
---

# Task Generation

You are generating implementation task files for autonomous LLM execution.

## Input

The calling skill will provide:
1. Layer name (e.g., "1-foundation" or "0-setup")
2. Layer plan (tasks assigned to this layer from plan-layers)
3. PRD analysis (from analyze-prd)
4. Template path (if any)
5. Output directory path
6. **Task batch** (optional): Subset of tasks to generate (for large layers)
7. **Previous review feedback** (optional): Critical issues from failed review attempt

### Batch Mode

When a task batch is provided:
- Only generate tasks listed in the batch
- Ignore other tasks in the layer plan
- Batch is a list of task IDs: `["L2-001", "L2-002", "L2-003", "L2-004", "L2-005"]`

### Retry Mode

When previous review feedback is provided, it will contain critical issues from the last attempt:

```json
{
  "attempt": 2,
  "previous_failures": [
    {
      "task_id": "L2-008",
      "issue": "Contains placeholder 'TBD' for schema field type"
    },
    {
      "task_id": "L2-009",
      "issue": "Missing interface methods in exports section"
    }
  ]
}
```

**CRITICAL**: When in retry mode, you MUST fix the reported issues. Do NOT repeat the same mistakes.

## CRITICAL CONSTRAINTS

These constraints are NON-NEGOTIABLE:

1. **Self-contained**: Each task file MUST contain ALL information needed. The implementing LLM cannot ask questions or look up external files.

2. **Small context**: Tasks are for ~50k token context models (Haiku, GLM 4.5-4.7). Be explicit, not clever.

3. **Interface contracts**: Dependencies use type signatures with imports, NOT full implementation code.

4. **Max 3 files**: Each task creates/modifies maximum 3 files (test file is additional).

5. **TDD approach**: Test requirements come BEFORE implementation. Tests are written first.

6. **No placeholders**: NEVER write "TODO", "TBD", "...", or any placeholder. If you don't know something, make a reasonable decision and document it.

7. **Concrete values**: Use specific names, paths, values. Never "appropriate" or "suitable".

## Task File Format

Generate XML files following this exact structure:

```xml
<task>
  <meta>
    <id>L1-001</id>
    <name>Create Project Database Model</name>
    <layer>1-foundation</layer>
    <priority>1</priority>
    <estimated-files>2</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    <!-- Copy RELEVANT PRD sections here -->
    <!-- Include feature description, acceptance criteria -->
    <!-- Do NOT copy the entire PRD -->
    </prd-excerpt>

    <tech-stack>
    Python 3.11, FastAPI 0.100+, SQLAlchemy 2.x, PostgreSQL 15
    </tech-stack>

    <template-base>webapps/backends/python</template-base>

    <project-structure>
    app/
      models/
        __init__.py
        base.py (exists - provides Base class)
      api/
      services/
    tests/
      models/
    </project-structure>
  </context>

  <dependencies>
    <interface name="Base" type="sqlalchemy-declarative-base">
    from sqlalchemy.orm import DeclarativeBase

    class Base(DeclarativeBase):
        """SQLAlchemy declarative base for all models."""
        pass
    </interface>

    <interface name="get_db" type="fastapi-dependency">
    from typing import Generator
    from sqlalchemy.orm import Session

    def get_db() -> Generator[Session, None, None]:
        """Yield database session. Auto-closes on request completion."""
        ...
    </interface>
  </dependencies>

  <objective>
  Create the SQLAlchemy model for storing PRD projects with support for
  project metadata and JSONB content storage.
  </objective>

  <requirements>
    <requirement id="1">
    Create model class `Project` in file `app/models/project.py`
    </requirement>

    <requirement id="2">
    Define enum `ProjectStatus` with values: draft, in_progress, complete
    </requirement>

    <requirement id="3">
    Project model fields:
    - id: UUID, primary key, server_default=uuid4
    - name: String(255), nullable=False
    - slug: String(100), unique=True, nullable=False
    - status: ProjectStatus enum, default=draft
    - prd_content: JSONB, nullable=True
    - created_at: DateTime, server_default=now()
    - updated_at: DateTime, onupdate=now()
    </requirement>

    <requirement id="4">
    Create Alembic migration `001_create_projects_table.py` that:
    - Creates `projects` table with all fields
    - Creates index on `slug` column
    - Creates index on `status` column
    </requirement>

    <requirement id="5">
    Export `Project` and `ProjectStatus` from `app/models/__init__.py`
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Create test file: `tests/models/test_project.py`
    </test>

    <test id="2">
    Test: test_create_project_with_valid_data
    - Create Project(name="Test Project", slug="test-project")
    - Assert project.id is a valid UUID
    - Assert project.status == ProjectStatus.draft
    - Assert project.created_at is not None
    </test>

    <test id="3">
    Test: test_project_slug_uniqueness
    - Create Project(name="First", slug="unique-slug")
    - Commit to database
    - Attempt to create Project(name="Second", slug="unique-slug")
    - Assert IntegrityError is raised on commit
    </test>

    <test id="4">
    Test: test_project_prd_content_json
    - Create Project with prd_content={"features": ["a", "b"]}
    - Commit and refresh from database
    - Assert project.prd_content == {"features": ["a", "b"]}
    - Assert type is dict
    </test>

    <test id="5">
    Test: test_project_status_enum
    - Create Project with status=ProjectStatus.in_progress
    - Commit and refresh
    - Assert project.status == ProjectStatus.in_progress
    - Assert project.status.value == "in_progress"
    </test>
  </test-requirements>

  <files-to-create>
    <file>app/models/project.py</file>
    <file>app/migrations/versions/001_create_projects_table.py</file>
    <file>tests/models/test_project.py</file>
  </files-to-create>

  <verification>
    <step>Run: `alembic upgrade head` - must complete without errors</step>
    <step>Run: `pytest tests/models/test_project.py -v` - all 4 tests pass</step>
    <step>Verify: `app/models/__init__.py` exports Project and ProjectStatus</step>
  </verification>

  <exports>
    <interface name="Project" type="sqlalchemy-model">
    from uuid import UUID
    from datetime import datetime
    from enum import Enum
    from sqlalchemy.orm import Mapped
    from app.models.base import Base

    class ProjectStatus(str, Enum):
        draft = "draft"
        in_progress = "in_progress"
        complete = "complete"

    class Project(Base):
        __tablename__ = "projects"

        id: Mapped[UUID]
        name: Mapped[str]
        slug: Mapped[str]  # unique
        status: Mapped[ProjectStatus]
        prd_content: Mapped[dict | None]
        created_at: Mapped[datetime]
        updated_at: Mapped[datetime]
    </interface>
  </exports>
</task>
```

## Generation Process

For each task in the layer plan:

1. **Read the task spec** from the layer plan
2. **Gather context**:
   - Relevant PRD excerpts
   - Dependencies from previous tasks (their `<exports>`)
   - Template patterns if applicable
3. **Write requirements** with exact specifications
4. **Write test requirements** with concrete test cases
5. **Define verification steps** with runnable commands
6. **Define exports** for downstream tasks

## Layer 0 Task Format

Layer 0 (setup) tasks are different - they use shell commands instead of code generation:

```xml
<task>
  <meta>
    <id>L0-001</id>
    <name>Copy Template to Target Directory</name>
    <layer>0-setup</layer>
    <priority>1</priority>
    <estimated-files>0</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Project type: greenfield
    Template: webapps/backends/python
    Target: /path/to/new-project
    </prd-excerpt>

    <tech-stack>
    Python 3.11, FastAPI, PostgreSQL (from template)
    </tech-stack>

    <template-source>webapps/backends/python</template-source>
    <target-directory>/path/to/new-project</target-directory>
  </context>

  <dependencies>
    <interface name="template" type="filesystem">
    Template exists at: webapps/backends/python
    Contains: Makefile, app/, frontend/, docker-compose.yml
    </interface>
  </dependencies>

  <objective>
  Copy the Python/FastAPI template to the target project directory,
  creating the foundation for the new project.
  </objective>

  <requirements>
    <requirement id="1">
    Create target directory if it doesn't exist: mkdir -p /path/to/new-project
    </requirement>
    <requirement id="2">
    Copy template contents: cp -r webapps/backends/python/* /path/to/new-project/
    </requirement>
    <requirement id="3">
    Remove template-specific files: rm -rf /path/to/new-project/.git (if exists)
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify directory exists: test -d /path/to/new-project
    </test>
    <test id="2">
    Verify Makefile exists: test -f /path/to/new-project/Makefile
    </test>
    <test id="3">
    Verify app directory: test -d /path/to/new-project/app
    </test>
  </test-requirements>

  <files-to-create>
    <!-- Layer 0 creates directories, not individual files -->
  </files-to-create>

  <verification>
    <step>Run: ls -la /path/to/new-project - must show template files</step>
    <step>Run: test -f /path/to/new-project/Makefile && echo "OK"</step>
  </verification>

  <exports>
    <interface name="project_directory" type="path">
    /path/to/new-project
    - Contains: app/, frontend/, Makefile, docker-compose.yml
    - Ready for: git init, dependency installation
    </interface>
  </exports>
</task>
```

## Quality Checks Before Output

Before writing each task file, verify:

- [ ] No "TODO", "TBD", "...", or placeholders
- [ ] All file paths are complete (not "in the models folder")
- [ ] All class/function names are specified
- [ ] All fields have types specified
- [ ] Test cases have concrete values
- [ ] Verification commands are runnable
- [ ] Interface contracts have imports

## Output

Write each task as a separate XML file:
- Filename: `{task_id}-{slug}.xml` (e.g., `L1-001-project-model.xml`)
- Location: `{output_dir}/{layer}/`

After writing all tasks, output a summary:
```
Generated {N} tasks for layer {layer}:
- L1-001-project-model.xml
- L1-002-conversation-model.xml
...
```
