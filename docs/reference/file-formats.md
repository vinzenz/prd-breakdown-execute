# File Formats Reference

Schemas for all file types used in the workflow.

---

## PRD Files

### index.md

Main PRD document in XML format.

```xml
<prd>
  <meta>
    <name>Project Name</name>
    <slug>project-slug</slug>
    <status>complete|in-progress</status>
    <created>2025-01-11</created>
    <updated>2025-01-11</updated>
  </meta>

  <overview>
    <problem-statement>...</problem-statement>
    <target-users>...</target-users>
    <value-proposition>...</value-proposition>
  </overview>

  <tech-stack>
    <project-type>greenfield|brownfield</project-type>
    <technologies>
      <technology name="Python" version="3.12" />
      <technology name="FastAPI" version="0.109" />
    </technologies>
    <rationale>...</rationale>
    <!-- brownfield only -->
    <integrations>...</integrations>
  </tech-stack>

  <features>
    <feature slug="feature-slug" priority="must-have|should-have|could-have|wont-have">
      Feature Name
    </feature>
  </features>

  <dependencies>
    <dependency name="package" version="^1.0">
      Purpose description
    </dependency>
  </dependencies>

  <!-- optional -->
  <motivation>...</motivation>
  <competitive-analysis>...</competitive-analysis>
  <non-functional-requirements>
    <performance>...</performance>
    <security>...</security>
    <scalability>...</scalability>
  </non-functional-requirements>
</prd>
```

### Feature Files

```xml
<feature>
  <meta>
    <name>Feature Name</name>
    <slug>feature-slug</slug>
    <priority>must-have</priority>
    <status>defined|tbd|in-progress</status>
  </meta>

  <description>
    Detailed feature description...
  </description>

  <acceptance-criteria>
    <criterion id="AC1">
      <given>Precondition</given>
      <when>Action</when>
      <then>Expected result</then>
    </criterion>
  </acceptance-criteria>

  <notes>
    Edge cases, considerations...
  </notes>
</feature>
```

### what-next.md

```xml
<progress>
  <status>complete|in-progress</status>

  <tbd-items>
    <item feature="feature-slug">
      What needs to be done
    </item>
  </tbd-items>

  <next-steps>
    What to work on when resuming
  </next-steps>

  <session-notes>
    Context about decisions made
  </session-notes>
</progress>
```

---

## Breakdown Files

### analysis.json

```json
{
  "prd_slug": "project-slug",
  "project_type": "greenfield|brownfield",
  "template": "webapps/backends/python",

  "features": [
    {
      "slug": "feature-slug",
      "name": "Feature Name",
      "priority": "must-have",
      "acceptance_criteria": [
        {
          "id": "AC1",
          "given": "...",
          "when": "...",
          "then": "..."
        }
      ]
    }
  ],

  "inferred_models": [
    {
      "name": "ModelName",
      "fields": [
        {
          "name": "field_name",
          "type": "str|int|UUID|datetime|...",
          "primary_key": true,
          "nullable": false,
          "max_length": 200
        }
      ],
      "relationships": [
        {
          "name": "relation_name",
          "target": "OtherModel",
          "type": "one-to-many|many-to-one|many-to-many"
        }
      ]
    }
  ],

  "inferred_endpoints": [
    {
      "method": "GET|POST|PUT|DELETE",
      "path": "/api/resource",
      "purpose": "Description"
    }
  ],

  "inferred_components": [
    {
      "name": "ComponentName",
      "type": "page|component|widget|form",
      "layer": 3
    }
  ],

  "dependencies": [
    {
      "name": "package-name",
      "version": "^1.0",
      "purpose": "Why needed"
    }
  ]
}
```

### layer_plan.json

```json
{
  "project_type": "greenfield|brownfield",
  "include_layer_0": true,

  "layers": {
    "0-setup": {
      "tasks": [
        {
          "id": "L0-001",
          "name": "Task Name",
          "files": ["file1.py", "file2.py"]
        }
      ],
      "dependencies": {}
    },
    "1-foundation": {
      "tasks": [...],
      "dependencies": {
        "L1-003": ["L1-001", "L1-002"]
      }
    }
  },

  "dependency_graph": {
    "L1-003": ["L1-001", "L1-002"],
    "L2-003": ["L2-001", "L2-002"]
  }
}
```

### manifest.json

```json
{
  "prd_slug": "project-slug",
  "generated_at": "2025-01-11T10:00:00Z",
  "total_tasks": 15,

  "layers": {
    "0-setup": {
      "status": "complete",
      "task_count": 3
    },
    "1-foundation": {
      "status": "complete",
      "task_count": 4
    }
  },

  "tasks": [
    {
      "id": "L0-001",
      "layer": "0-setup",
      "name": "Task Name",
      "file": "0-setup/L0-001-task-name.xml"
    }
  ]
}
```

---

## Task Files

### Task XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>L2-003</id>
    <name>Task Name</name>
    <layer>2-backend</layer>
    <priority>1</priority>
    <estimated-files>2</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
      Relevant PRD content...
    </prd-excerpt>
    <tech-stack>
      Python 3.12, FastAPI 0.109, SQLAlchemy 2.0
    </tech-stack>
    <template-base>webapps/backends/python</template-base>
    <project-structure>
      src/
      ├── models/
      └── api/
    </project-structure>
  </context>

  <dependencies>
    <interface name="InterfaceName" type="type-category">
      from module import Something

      class InterfaceName:
          def method(self, arg: Type) -> ReturnType: ...
    </interface>
  </dependencies>

  <objective>
    What this task achieves (1-3 sentences).
  </objective>

  <requirements>
    <requirement id="R1">
      Detailed specification...
    </requirement>
    <requirement id="R2">
      Another specification...
    </requirement>
  </requirements>

  <test-requirements>
    <test id="T1">
      File: tests/path/test_file.py

      Test: test_function_name
      Setup: Preconditions
      Action: What to do
      Assert: Expected outcomes
    </test>
  </test-requirements>

  <files-to-create>
    <file>src/path/file1.py</file>
    <file>src/path/file2.py</file>
  </files-to-create>

  <verification>
    <step>Run: pytest tests/path/test_file.py -v</step>
    <step>Verify: All tests pass</step>
    <step>Check: Some condition</step>
  </verification>

  <exports>
    <interface name="ExportName" type="type-category">
      from module import Something

      class ExportName: ...
    </interface>
  </exports>
</task>
```

---

## Execution Files

### execute-state.json

```json
{
  "schema_version": "2.0",

  "prd_slug": "project-slug",
  "project_path": "/path/to/project",
  "worktree_dir": "/path/to/project/.worktrees",
  "tasks_path": "/path/to/docs/tasks/project-slug",

  "status": "pending|in_progress|completed|stopped|abandoned",
  "current_layer": "2-backend",
  "current_batch": 2,

  "started_at": "2025-01-11T10:00:00Z",
  "updated_at": "2025-01-11T10:15:00Z",
  "completed_at": null,

  "layers": {
    "0-setup": {
      "status": "pending|in_progress|completed",
      "tasks_total": 4,
      "tasks_completed": 4,
      "tasks_failed": 0,
      "started_at": "2025-01-11T10:00:00Z",
      "completed_at": "2025-01-11T10:03:00Z"
    }
  },

  "tasks": {
    "L2-001": {
      "status": "pending|in_progress|verifying|verified|completed|failed|abandoned",
      "attempts": 1,
      "worktree_path": "/path/to/.worktrees/L2-001",
      "branch": "worktree-L2-001",
      "commits": [
        {
          "hash": "abc1234",
          "type": "implementation|fix",
          "attempt": 1,
          "created_at": "2025-01-11T10:05:00Z"
        }
      ],
      "errors": [
        {
          "attempt": 1,
          "message": "Error description",
          "timestamp": "2025-01-11T10:05:00Z"
        }
      ],
      "retry_feedback": [
        {
          "attempt": 2,
          "feedback": "Actionable fix suggestion"
        }
      ],
      "merged_at": "2025-01-11T10:06:00Z"
    }
  },

  "merge_queue": [
    {
      "task_id": "L2-001",
      "priority": 1,
      "status": "pending|ready|merging|merged"
    }
  ],

  "metrics": {
    "total_tasks": 15,
    "completed_tasks": 11,
    "failed_tasks": 0,
    "total_attempts": 13,
    "total_commits": 14,
    "elapsed_time_seconds": 900
  }
}
```

---

## Interface Types

Common type values for interface contracts:

| Type | Description |
|------|-------------|
| `sqlalchemy-model` | SQLAlchemy ORM model |
| `pydantic-model` | Pydantic schema |
| `repository` | Data access class |
| `service` | Business logic class |
| `fastapi-router` | FastAPI router |
| `enum` | Enumeration type |
| `function` | Standalone function |

---

## Next Steps

- [Commands Reference](commands.md)
- [Task Format Details](../skills/breakdown/task-format.md)
- [State Management](../skills/execute/state.md)
