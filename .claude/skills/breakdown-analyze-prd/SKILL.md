---
name: breakdown-analyze-prd
description: Analyze a PRD to extract features, tech stack, and implementation requirements. Called by /breakdown skill during Phase 2.
context: fork
allowed-tools: Read Glob Grep
model: claude-haiku-4-5
---

# PRD Analysis

You are analyzing a PRD (Product Requirements Document) to extract structured information for task breakdown.

## Input

The calling skill will provide the full PRD XML content.

## Your Task

Extract and structure the following information from the PRD:

### 1. Features

For each feature in `<features>`:
- Name
- Priority (must-have, should-have, could-have, won't-have)
- Summary
- File path to detailed spec (if exists)

### 2. Tech Stack

From `<tech-stack>`:
- **Project type**: Extract from `<type>` element (greenfield/brownfield)
- **Project path**: Extract from `<project-path>` element (brownfield only)
- All technologies with versions from `<selected>` element
- Template path: Detect from technologies (e.g., "Python + FastAPI" → `webapps/backends/python`)
- Rationale for choices from `<rationale>` element

### 3. Data Models (Inferred)

Analyze features to identify implied database models:
- Model name
- Key fields (inferred from feature descriptions)
- Relationships between models

### 4. API Endpoints (Inferred)

Analyze features to identify implied API endpoints:
- HTTP method
- Path
- Purpose
- Related feature

### 5. Frontend Components (Inferred)

Analyze features and UI layout to identify:
- Component name
- Type (page, layout, widget, form, etc.)
- Related feature

### 6. Dependencies

From `<dependencies>`:
- Name
- Version
- Purpose

### 7. Template Information

If tech-stack references a template:
- Template path
- What the template provides (auth, database setup, etc.)

## Output Format

Return a JSON object with this structure:

```json
{
  "prd_meta": {
    "name": "Project Name",
    "slug": "project-slug",
    "status": "complete|in-progress"
  },
  "features": [
    {
      "name": "Feature Name",
      "priority": "must-have",
      "summary": "Brief description",
      "spec_file": "features/feature-slug.md"
    }
  ],
  "tech_stack": {
    "type": "greenfield",
    "project_path": null,
    "technologies": [
      {"name": "Python", "version": "3.11", "purpose": "Backend language"},
      {"name": "FastAPI", "version": "0.100+", "purpose": "API framework"}
    ],
    "template": {
      "path": "webapps/backends/python",
      "provides": ["auth", "database", "docker"],
      "detected_from": "Python + FastAPI in selected technologies"
    }
  },
  "data_models": [
    {
      "name": "Project",
      "fields": ["id", "name", "slug", "status", "content"],
      "relationships": ["has_many: Conversation"]
    }
  ],
  "api_endpoints": [
    {
      "method": "POST",
      "path": "/api/projects",
      "purpose": "Create new project",
      "feature": "Project Management"
    }
  ],
  "frontend_components": [
    {
      "name": "Dashboard",
      "type": "page",
      "feature": "Dashboard"
    }
  ],
  "dependencies": [
    {
      "name": "anthropic",
      "version": "latest",
      "purpose": "LLM API client"
    }
  ]
}
```

## Template Detection

Detect the template from `<tech-stack><selected>` text using these mappings:

| Tech Stack Keywords | Template Path |
|---------------------|---------------|
| Python, FastAPI | `webapps/backends/python` |
| Go, Chi | `webapps/backends/go` |
| TanStack, TanStack Start | `webapps/backends/tanstack` |

If template is explicitly mentioned in the PRD (e.g., "from webapp template"), use that.
If no match found, set `template.path` to `null` and note in `detected_from`.

## Analysis Guidelines

1. **Be thorough**: Capture all features, not just the obvious ones
2. **Infer carefully**: Data models and APIs should be reasonable inferences, not guesses
3. **Use PRD language**: Match names and terminology from the PRD
4. **Note uncertainties**: If something is unclear, include it with a note
5. **Brownfield**: If `<type>brownfield</type>`, expect `<project-path>` to be present

## Example Inference

Given feature:
```xml
<feature priority="must-have">
  <name>Project Management</name>
  <summary>Create, switch, and list PRD projects with status visibility</summary>
</feature>
```

Infer:
- Data model: `Project` with fields (id, name, status, created_at)
- API endpoints: GET /projects, POST /projects, GET /projects/:id, PUT /projects/:id
- Frontend: ProjectList component, ProjectCard component

## Do NOT

- Make up features not in the PRD
- Guess at specific implementation details
- Include your own opinions about architecture
- Truncate or summarize features

Return the complete JSON analysis.
