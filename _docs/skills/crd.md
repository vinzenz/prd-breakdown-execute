---
layout: default
title: /crd Command
parent: Skills Reference
nav_order: 2
---

# /crd Command

The `/crd` command creates Change Request Documents for modifying existing codebases.

## Usage

```bash
claude /crd
```

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                         /crd Workflow                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Check for PROJECT.md                                        │
│     │                                                            │
│     ├─► Missing? Run crd-investigate to generate                │
│     │                                                            │
│     └─► Exists but stale? Run crd-context-update                │
│                                                                  │
│  2. Capture change request from user                            │
│     │                                                            │
│     └─► What change do you want to make?                        │
│                                                                  │
│  3. Run impact analysis                                         │
│     │                                                            │
│     └─► Which files/features/APIs are affected?                 │
│                                                                  │
│  4. Generate CRD document                                       │
│     │                                                            │
│     └─► docs/crd/{slug}/index.xml                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## PROJECT.md Context

Before creating a CRD, the system ensures codebase context exists:

### If PROJECT.md is Missing

The `crd-investigate` agent performs deep analysis:

```
Analyzing codebase...
├── Scanning directory structure
├── Reading key files (package.json, go.mod, etc.)
├── Identifying architecture patterns
├── Mapping features and APIs
├── Detecting database schemas
└── Generating PROJECT.md

PROJECT.md created with:
- Architecture overview
- Tech stack details
- Feature registry
- API endpoints
- Database schemas
```

### If PROJECT.md is Stale

The `crd-context-update` agent performs incremental update:

```bash
git diff HEAD~50 --name-only  # Check recent changes
```

```
Updating PROJECT.md...
├── New files detected: src/api/reports.py
├── Modified APIs: /users endpoint changed
├── New schema: Report model added
└── Updated PROJECT.md sections
```

## Change Request Capture

The wizard captures your change intent:

```
What change would you like to make?

> Add a reporting feature that generates PDF reports
  of library activity - checkouts, returns, and fees
  collected per time period.

What type of change is this?
  [1] feature-add - New functionality
  [2] feature-modify - Change existing feature
  [3] feature-remove - Remove functionality
  [4] refactor - Improve without changing behavior
> 1

Priority:
  [1] Must have (critical path)
  [2] Should have (important)
  [3] Could have (nice to have)
> 2
```

## Impact Analysis

The `crd-impact-analysis` agent identifies affected areas:

```
Running impact analysis...

Affected Files:
├── src/models/ - New Report model needed
├── src/api/ - New reports.py endpoint
├── src/services/ - New report_generator.py
└── tests/ - New test files

Affected Features:
├── Admin Dashboard - Will consume reports
└── Late Fees - Data source for reports

Affected APIs:
├── GET /admin/reports - New endpoint
├── POST /admin/reports/generate - New endpoint
└── GET /fees - Existing, used as data source

Schema Changes:
└── New Report table with foreign keys to User, Book
```

## CRD Document Format

Output: `docs/crd/pdf-reports/index.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<change-request>
  <meta>
    <slug>pdf-reports</slug>
    <type>feature-add</type>
    <status>draft</status>
    <created>2025-01-15T10:30:00Z</created>
  </meta>

  <context>
    <project-context-ref>PROJECT.md</project-context-ref>
    <related-features>
      <feature>admin-dashboard</feature>
      <feature>late-fees</feature>
    </related-features>
  </context>

  <change-request>
    <summary>Add PDF report generation for library activity</summary>
    <motivation>
      Administrators need periodic reports on library usage
      for board meetings and compliance purposes.
    </motivation>
    <scope>
      - Report generation service
      - PDF rendering
      - Admin API endpoints
      - Dashboard integration
    </scope>
  </change-request>

  <impact-analysis>
    <affected-files>
      <file action="create">src/models/report.py</file>
      <file action="create">src/api/reports.py</file>
      <file action="create">src/services/report_generator.py</file>
      <file action="modify">src/api/admin.py</file>
    </affected-files>
    <affected-features>
      <feature impact="extends">admin-dashboard</feature>
      <feature impact="uses">late-fees</feature>
    </affected-features>
    <affected-apis>
      <api action="create">GET /admin/reports</api>
      <api action="create">POST /admin/reports/generate</api>
    </affected-apis>
    <schema-changes>
      <table action="create">reports</table>
    </schema-changes>
  </impact-analysis>

  <requirements priority="must">
    <requirement id="R1">Generate PDF reports of checkouts by date range</requirement>
    <requirement id="R2">Generate PDF reports of late fees collected</requirement>
    <requirement id="R3">Admin-only access to report endpoints</requirement>
  </requirements>

  <requirements priority="should">
    <requirement id="R4">Schedule automated report generation</requirement>
    <requirement id="R5">Email reports to administrators</requirement>
  </requirements>

  <acceptance-criteria>
    <criterion id="AC1">
      <given>Admin is authenticated</given>
      <when>Admin requests checkout report for date range</when>
      <then>PDF is generated with all checkouts in range</then>
    </criterion>
    <criterion id="AC2">
      <given>User is not admin</given>
      <when>User tries to access report endpoints</when>
      <then>403 Forbidden is returned</then>
    </criterion>
  </acceptance-criteria>
</change-request>
```

## Next Steps

After creating your CRD:

```bash
# Break down into tasks
claude /breakdown docs/crd/pdf-reports/

# Execute the tasks
claude /execute docs/crd/pdf-reports/tasks/
```

The breakdown phase uses impact analysis to scope tasks appropriately.

After execution, `PROJECT.md` is automatically updated with:
- New features implemented
- New API endpoints
- New database schemas

## Related Commands

- [`/crd-context`]({{ '/docs/skills/crd-context/' | relative_url }}) - Manage PROJECT.md independently
