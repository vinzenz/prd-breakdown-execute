# Task File Format Specification

This document defines the XML schema for implementation task files.

## Root Element

```xml
<task>
  <!-- All sections below are REQUIRED unless marked optional -->
</task>
```

## Sections

### 1. Meta (Required)

Task identification and classification.

```xml
<meta>
  <id>L1-001</id>                    <!-- Layer number + sequence: L{layer}-{seq} -->
  <name>Create Project Model</name>   <!-- Human-readable task name -->
  <layer>1-foundation</layer>         <!-- Layer identifier -->
  <priority>1</priority>              <!-- Execution order within layer (1 = first) -->
  <estimated-files>2</estimated-files> <!-- Number of files to create/modify -->
</meta>
```

**Constraints:**
- `id`: Must match pattern `L[1-4]-[0-9]{3}`
- `layer`: One of: `1-foundation`, `2-backend`, `3-frontend`, `4-integration`
- `priority`: Integer 1-99
- `estimated-files`: Integer 1-3 (max 3 files per task)

### 2. Context (Required)

All background information needed to understand the task.

```xml
<context>
  <prd-excerpt>
  <!-- Copy relevant PRD sections here -->
  <!-- Include feature description, acceptance criteria, etc. -->
  </prd-excerpt>

  <tech-stack>
  <!-- List technologies and versions -->
  Python 3.11, FastAPI 0.100+, SQLAlchemy 2.x, PostgreSQL 15
  </tech-stack>

  <template-base>webapps/backends/python</template-base>  <!-- Optional: template path -->

  <project-structure>
  <!-- Optional: Relevant directory structure -->
  app/
    models/
    api/
    services/
  </project-structure>
</context>
```

**Constraints:**
- `prd-excerpt`: Must include all relevant requirements from PRD
- `tech-stack`: Must specify versions where known
- Do NOT reference external files - all context must be inline

### 3. Dependencies (Required)

Interface contracts this task depends on.

```xml
<dependencies>
  <interface name="Base" type="sqlalchemy-declarative-base">
  from sqlalchemy.orm import DeclarativeBase

  class Base(DeclarativeBase):
      pass
  </interface>

  <interface name="get_db" type="fastapi-dependency">
  from typing import Generator
  from sqlalchemy.orm import Session

  def get_db() -> Generator[Session, None, None]:
      """Yields database session, auto-closes on completion."""
      ...
  </interface>

  <!-- Add all interfaces this task needs -->
</dependencies>
```

**Constraints:**
- Each `<interface>` must have `name` and `type` attributes
- Include complete type signatures with imports
- Include docstrings for functions
- Do NOT include implementation details unless critical

### 4. Objective (Required)

Clear, concise description of what this task accomplishes.

```xml
<objective>
Create the SQLAlchemy model for storing PRD projects. The model should
support storing project metadata (name, slug, status) and the full PRD
content as JSONB for flexible schema.
</objective>
```

**Constraints:**
- 1-3 sentences
- Focus on WHAT, not HOW
- No implementation details

### 5. Requirements (Required)

Specific, verifiable requirements.

```xml
<requirements>
  <requirement id="1">
  Model class named `Project` in file `app/models/project.py`
  </requirement>

  <requirement id="2">
  Fields:
  - id: UUID, primary key, auto-generated
  - name: str, max 255 chars, required
  - slug: str, max 100 chars, unique, required
  - status: ProjectStatus enum (draft, in_progress, complete)
  - prd_content: JSONB, nullable
  - created_at: datetime, auto-set on create
  - updated_at: datetime, auto-set on update
  </requirement>

  <requirement id="3">
  Alembic migration file that creates the `projects` table
  </requirement>

  <requirement id="4">
  Export Project class in `app/models/__init__.py`
  </requirement>
</requirements>
```

**Constraints:**
- Each requirement has unique `id` attribute
- Requirements must be specific and verifiable
- No ambiguous terms like "appropriate", "good", "proper"
- Include exact file paths, class names, field names

### 6. Test Requirements (Required)

Tests to write BEFORE implementation (TDD).

```xml
<test-requirements>
  <test id="1">
  Test file: `tests/models/test_project.py`
  </test>

  <test id="2">
  Test: Project can be created with valid name and slug
  - Create Project(name="Test", slug="test")
  - Assert project.id is UUID
  - Assert project.created_at is set
  </test>

  <test id="3">
  Test: Slug uniqueness is enforced
  - Create Project with slug="unique"
  - Attempt to create another with same slug
  - Assert IntegrityError is raised
  </test>

  <test id="4">
  Test: prd_content stores and retrieves dict correctly
  - Create Project with prd_content={"key": "value"}
  - Retrieve from DB
  - Assert retrieved.prd_content == {"key": "value"}
  </test>

  <test id="5">
  Test: Status enum works correctly
  - Create Project with status=ProjectStatus.draft
  - Assert stored value is correct
  </test>
</test-requirements>
```

**Constraints:**
- Tests are written FIRST (TDD)
- Each test has unique `id` attribute
- Test descriptions include:
  - Setup steps
  - Action to take
  - Expected assertion
- Use concrete values, not "some value" or "any value"

### 7. Files to Create (Required)

Explicit list of files this task creates or modifies.

```xml
<files-to-create>
  <file>app/models/project.py</file>
  <file>app/models/__init__.py</file>
  <file>app/migrations/versions/001_create_projects_table.py</file>
  <file>tests/models/test_project.py</file>
</files-to-create>
```

**Constraints:**
- Maximum 3 files (excluding test file)
- Use full relative paths from project root
- List in order of creation

### 8. Verification (Required)

Commands to verify task completion.

```xml
<verification>
  <step>Run: `alembic upgrade head` - migration applies without error</step>
  <step>Run: `pytest tests/models/test_project.py -v` - all tests pass</step>
  <step>Check: `app/models/__init__.py` exports Project class</step>
</verification>
```

**Constraints:**
- Each step is a runnable command or checkable condition
- Include expected outcome
- Steps should be executable in order

### 9. Exports (Required)

Interface contracts this task provides for later tasks.

```xml
<exports>
  <interface name="Project" type="sqlalchemy-model">
  from uuid import UUID
  from datetime import datetime
  from enum import Enum
  from sqlalchemy.orm import Mapped

  class ProjectStatus(Enum):
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
```

**Constraints:**
- Include all public interfaces created by this task
- Use complete type annotations
- Include imports needed to use the interface
- This becomes the dependency for downstream tasks

## Validation Rules

1. **No placeholders**: No "TODO", "TBD", "...", or "[fill in]"
2. **No external references**: All information must be in the task file
3. **Concrete values**: Use specific names, paths, values - not "appropriate" or "suitable"
4. **Complete types**: All interfaces must have full type annotations
5. **Runnable verification**: All verification steps must be executable commands
