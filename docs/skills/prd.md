# /prd Command Reference

Interactive Product Requirements Document creation.

---

## Overview

The `/prd` command guides you through an 8-phase workflow to create a complete, structured PRD that can be processed by the `/breakdown` skill.

```bash
claude /prd
```

---

## Arguments

| Argument | Description |
|----------|-------------|
| `--resume` | Resume an incomplete PRD |
| `--output-dir <path>` | Custom output directory (default: `docs/prd/`) |

---

## The 8 Phases

### Phase 1: Idea Capture

**Purpose:** Understand the core problem and value proposition.

**Questions asked:**
- What problem are you solving?
- Who are your target users?
- What's the core value proposition?

**Adaptive follow-up:** Claude asks more questions for vague answers, moves forward quickly for clear ones.

### Phase 2: Tech Stack

**Purpose:** Define the technical foundation.

**For greenfield projects:**
- What technologies do you prefer?
- Claude suggests 2-3 stack options with tradeoffs
- You select or customize

**For brownfield projects:**
- What existing technologies are in use?
- What are the integration points?
- Any data migration needs?

**Output:** Technology table with rationale

### Phase 3: Features (MoSCoW)

**Purpose:** Define and prioritize features.

**Prioritization:**
- **Must-Have:** Essential for MVP
- **Should-Have:** Important, can add shortly after
- **Could-Have:** Nice to have
- **Won't-Have:** Explicitly out of scope

**For each feature:**
- Name and description
- Acceptance criteria (Given/When/Then)
- Status: `defined` or `tbd`

### Phase 4: Dependencies

**Purpose:** Document external requirements.

**For each dependency:**
- Name
- Version requirements
- Purpose/usage

### Phase 5-6: Optional Sections

**Available sections:**
- Motivation & Use Cases
- Competitive Analysis
- Non-Functional Requirements (performance, security, scalability)

**Validation:** Claude checks for contradictions and tech-feature compatibility.

### Phase 7: Interactive Review

**Purpose:** Review and revise before finalizing.

**Actions:**
- View summary of each section
- Request revisions
- Add missing details

### Phase 8: Output Generation

**Purpose:** Create the final PRD files.

**Creates:**
```
docs/prd/{project-slug}/
├── index.md          # Main PRD document
├── what-next.md      # Progress tracking
└── features/
    ├── feature-1.md  # Feature details
    └── feature-2.md
```

---

## Output Format

### index.md Structure

```xml
<prd>
  <meta>
    <name>Project Name</name>
    <slug>project-slug</slug>
    <status>complete</status>
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
    <!-- For brownfield: -->
    <integrations>...</integrations>
  </tech-stack>

  <features>
    <feature slug="feature-1" priority="must-have">
      Feature Name
    </feature>
    <!-- Links to individual feature files -->
  </features>

  <dependencies>
    <dependency name="package-name" version="^1.0">
      Purpose description
    </dependency>
  </dependencies>

  <!-- Optional sections -->
  <motivation>...</motivation>
  <competitive-analysis>...</competitive-analysis>
  <non-functional-requirements>
    <performance>...</performance>
    <security>...</security>
  </non-functional-requirements>
</prd>
```

### Feature File Structure

```xml
<feature>
  <meta>
    <name>Feature Name</name>
    <slug>feature-slug</slug>
    <priority>must-have</priority>
    <status>defined</status>
  </meta>

  <description>
    Detailed feature description...
  </description>

  <acceptance-criteria>
    <criterion id="AC1">
      <given>A user is logged in</given>
      <when>They perform an action</when>
      <then>Something happens</then>
    </criterion>
  </acceptance-criteria>

  <notes>
    Edge cases, considerations...
  </notes>
</feature>
```

### what-next.md Structure

```xml
<progress>
  <status>complete|in-progress</status>

  <tbd-items>
    <!-- Features or sections marked for later -->
    <item feature="feature-slug">
      Acceptance criteria needed
    </item>
  </tbd-items>

  <next-steps>
    What to work on when resuming
  </next-steps>

  <session-notes>
    Context about decisions made, alternatives considered
  </session-notes>
</progress>
```

---

## Resume Capability

If you stop mid-PRD:

```bash
claude /prd --resume
```

```
Found incomplete PRDs:
  1. slack-task-manager (Phase 5)
  2. analytics-dashboard (Phase 3)

Which would you like to resume?
```

Resume reads from `what-next.md` to restore context.

---

## Tips for Great PRDs

### Be Specific

| Instead of | Say |
|------------|-----|
| "Fast" | "Response time < 200ms p95" |
| "Secure" | "OAuth 2.0, encrypted at rest" |
| "Easy to use" | "Max 3 clicks to complete task" |

### Think About Edge Cases

- What happens when there's no data?
- What if the external service is down?
- How do errors get communicated?

### Write Clear Acceptance Criteria

```
Good:
  Given a user has 3 incomplete tasks
  When they mark one as complete
  Then the task list shows 2 incomplete tasks

Bad:
  Given a user
  When they do something
  Then it works
```

### Prioritize Ruthlessly

Must-have should be minimal viable:
- Can you launch without this feature?
- If yes, it's not must-have

---

## Examples

### Simple API Project

```bash
claude /prd
```

```
Problem: Need a REST API for managing inventory
Users: Internal operations team
Value: Real-time inventory tracking

Tech: Python + FastAPI (greenfield)

Features (must-have):
  - Product CRUD
  - Stock level tracking
  - Low stock alerts
```

### Integration Project

```bash
claude /prd
```

```
Problem: Add analytics to existing dashboard
Users: Same users as current dashboard
Value: Data-driven decisions

Tech: Brownfield, existing React + Node stack

Features (must-have):
  - Usage metrics collection
  - Dashboard widgets
  - Export to CSV
```

---

## Next Steps

After creating your PRD:

```bash
claude /breakdown docs/prd/{project-slug}/index.md
```

See: [/breakdown Reference](breakdown/README.md)
