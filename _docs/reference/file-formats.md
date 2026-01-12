---
layout: default
title: File Formats
parent: Reference
nav_order: 2
---

# File Formats

Schema documentation for XML and JSON files.

## Task XML

Self-contained task specification.

### Schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>string</id>           <!-- Unique: L{layer}-{seq}-{slug} -->
    <name>string</name>        <!-- Human-readable name -->
    <layer>integer</layer>     <!-- 0-4 -->
    <priority>integer</priority> <!-- Within layer -->
  </meta>

  <context>
    <prd-excerpt>string</prd-excerpt>   <!-- Relevant PRD content -->
    <tech-stack>string</tech-stack>      <!-- Framework details -->
  </context>

  <dependencies>
    <dependency task="string">           <!-- Task ID -->
      <interface-contract>string</interface-contract>
    </dependency>
  </dependencies>

  <objective>string</objective>          <!-- What to achieve -->

  <requirements>
    <requirement id="string">string</requirement>
  </requirements>

  <test-requirements>
    <test id="string">string</test>
  </test-requirements>

  <files-to-create>
    <file>string</file>                  <!-- Max 3 files -->
  </files-to-create>

  <verification>
    <command>string</command>            <!-- Runnable command -->
  </verification>

  <exports>
    <export name="string">string</export> <!-- Interface contract -->
  </exports>
</task>
```

### Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>L2-001-auth-api</id>
    <name>Authentication API</name>
    <layer>2</layer>
    <priority>1</priority>
  </meta>

  <context>
    <prd-excerpt>JWT authentication with signup, login, logout.</prd-excerpt>
    <tech-stack>FastAPI, python-jose, passlib</tech-stack>
  </context>

  <dependencies>
    <dependency task="L1-001-user-model">
      <interface-contract>
class User(Base):
    id: UUID
    email: str
    hashed_password: str
      </interface-contract>
    </dependency>
  </dependencies>

  <objective>Implement auth endpoints returning JWT tokens.</objective>

  <requirements>
    <requirement id="R1">POST /auth/signup returns 201</requirement>
    <requirement id="R2">POST /auth/login returns JWT</requirement>
  </requirements>

  <test-requirements>
    <test id="T1">test_signup_success</test>
    <test id="T2">test_login_returns_jwt</test>
  </test-requirements>

  <files-to-create>
    <file>src/api/auth.py</file>
    <file>tests/api/test_auth.py</file>
  </files-to-create>

  <verification>
    <command>pytest tests/api/test_auth.py -v</command>
  </verification>

  <exports>
    <export name="auth_router">
from fastapi import APIRouter
auth_router = APIRouter(prefix="/auth")
    </export>
  </exports>
</task>
```

---

## CRD XML

Change Request Document for brownfield projects.

### Schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<change-request>
  <meta>
    <slug>string</slug>
    <type>feature-add|feature-modify|feature-remove|refactor</type>
    <status>draft|approved|completed</status>
    <created>datetime</created>
  </meta>

  <context>
    <project-context-ref>string</project-context-ref>
    <related-features>
      <feature>string</feature>
    </related-features>
  </context>

  <change-request>
    <summary>string</summary>
    <motivation>string</motivation>
    <scope>string</scope>
  </change-request>

  <impact-analysis>
    <affected-files>
      <file action="create|modify|delete">string</file>
    </affected-files>
    <affected-features>
      <feature impact="extends|modifies|removes|uses">string</feature>
    </affected-features>
    <affected-apis>
      <api action="create|modify|delete">string</api>
    </affected-apis>
    <schema-changes>
      <table action="create|modify|delete">string</table>
    </schema-changes>
  </impact-analysis>

  <requirements priority="must|should|could">
    <requirement id="string">string</requirement>
  </requirements>

  <acceptance-criteria>
    <criterion id="string">
      <given>string</given>
      <when>string</when>
      <then>string</then>
    </criterion>
  </acceptance-criteria>
</change-request>
```

---

## analysis.json

PRD/CRD analysis output.

### Schema

```json
{
  "source": "string (path to PRD/CRD)",
  "analyzed_at": "datetime",
  "entities": [
    {
      "name": "string",
      "fields": ["string"],
      "relationships": [
        {
          "target": "string",
          "type": "belongs_to|has_many|has_one"
        }
      ]
    }
  ],
  "features": [
    {
      "name": "string",
      "requirements": ["string"],
      "endpoints": ["string"]
    }
  ],
  "tech_stack": {
    "framework": "string",
    "database": "string",
    "testing": "string"
  }
}
```

---

## layer_plan.json

Layer organization and dependencies.

### Schema

```json
{
  "generated_at": "datetime",
  "layers": {
    "0": {
      "name": "setup",
      "description": "string",
      "tasks": ["string (task IDs)"]
    },
    "1": {
      "name": "foundation",
      "description": "string",
      "tasks": ["string"]
    }
  },
  "dependencies": {
    "L2-001": ["L1-001", "L1-002"]
  }
}
```

---

## manifest.json

Task inventory and metadata.

### Schema

```json
{
  "source": "string (path to PRD/CRD)",
  "generated_at": "datetime",
  "total_tasks": "integer",
  "layers": {
    "0": ["L0-001-task-name"],
    "1": ["L1-001-task-name", "L1-002-task-name"]
  },
  "tasks": {
    "L0-001-task-name": {
      "layer": 0,
      "priority": 1,
      "files": ["string"],
      "dependencies": []
    }
  }
}
```

---

## execute-state.json

Execution progress tracking.

### Schema

```json
{
  "status": "pending|in_progress|completed|failed",
  "started_at": "datetime",
  "completed_at": "datetime|null",
  "current_layer": "integer",
  "tasks": {
    "L1-001-task-name": {
      "status": "pending|in_progress|completed|failed",
      "attempts": "integer",
      "started_at": "datetime|null",
      "completed_at": "datetime|null",
      "last_error": "string|null",
      "worktree": "string|null"
    }
  }
}
```

---

## PROJECT.md

Brownfield codebase context (dual-format).

### Structure

```markdown
# Project Name

## Overview
Human-readable project description.

## Architecture
Description of the system architecture.

## Tech Stack
- Framework: X
- Database: Y
- Testing: Z

## Key Patterns
Description of patterns used.

<project-context>
  <features>
    <feature name="auth">
      <description>User authentication</description>
      <files>
        <file>src/api/auth.py</file>
      </files>
      <signatures>
        <signature name="login">
async def login(email: str, password: str) -> Token
        </signature>
      </signatures>
    </feature>
  </features>

  <api-registry>
    <endpoint method="POST" path="/auth/login">
      <description>User login</description>
      <request>{"email": "string", "password": "string"}</request>
      <response>{"token": "string"}</response>
    </endpoint>
  </api-registry>

  <schema-registry>
    <table name="users">
      <column name="id" type="UUID" primary="true"/>
      <column name="email" type="VARCHAR" unique="true"/>
    </table>
  </schema-registry>
</project-context>
```
