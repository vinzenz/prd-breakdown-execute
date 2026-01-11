# Creating Your First PRD

A detailed walkthrough of the `/prd` command.

---

## Starting the PRD

```bash
claude /prd
```

You'll be greeted with:

```
Welcome to PRD creation!

I'll guide you through 8 phases to create a complete
Product Requirements Document for your project.

Let's start with your idea.
```

---

## Phase 1: Idea Capture

The first phase focuses on understanding your idea deeply.

```
What problem are you solving?
─────────────────────────────
Describe the problem in 2-3 sentences.
```

**Good answer:**
> Teams struggle to track tasks across multiple tools. They waste time
> switching between Slack, spreadsheets, and email. We need a unified
> task view that lives where they already work - Slack.

**What happens:** Claude will ask follow-up questions based on your answer quality. Vague answers get more probing; clear answers move forward quickly.

```
Who are your target users?
──────────────────────────
Be specific about who will use this.
```

**Good answer:**
> Small development teams of 5-15 people who use Slack as their primary
> communication tool. They're comfortable with technology but don't want
> another complex tool to learn.

```
What's the core value proposition?
──────────────────────────────────
In one sentence, why would someone use this?
```

**Good answer:**
> Simple task tracking without leaving Slack - create, assign, and
> complete tasks through natural conversation.

---

## Phase 2: Tech Stack

```
Is this a new project or integration into existing code?
────────────────────────────────────────────────────────
1. Greenfield (new project)
2. Brownfield (existing project)
```

For a new project:

```
What technologies do you prefer?
────────────────────────────────
I'll suggest a stack based on your answer.
```

**Example answer:**
> Python for backend, React for frontend if needed.
> I want something that's easy to deploy to Heroku or Railway.

**Claude suggests:**
```
Based on your requirements, I recommend:

┌─────────────────────────────────────────────────────────────┐
│ Stack Option A (Recommended)                                │
├─────────────────────────────────────────────────────────────┤
│ Backend:  Python 3.12 + FastAPI                             │
│ Database: PostgreSQL                                        │
│ ORM:      SQLAlchemy 2.0 with async                        │
│ Deploy:   Railway or Heroku                                │
│ Reason:   Fast development, excellent async support,       │
│           great documentation, easy deployment             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Stack Option B                                              │
├─────────────────────────────────────────────────────────────┤
│ Backend:  Go + Chi                                         │
│ Database: PostgreSQL                                        │
│ ORM:      sqlc (code generation)                           │
│ Deploy:   Single binary, any platform                      │
│ Reason:   Better performance, type safety, smaller memory   │
└─────────────────────────────────────────────────────────────┘

Which would you prefer?
```

---

## Phase 3: Features (MoSCoW)

```
Let's define features using MoSCoW prioritization:
─────────────────────────────────────────────────

Must-Have: Essential for MVP, cannot launch without
Should-Have: Important but can be added shortly after
Could-Have: Nice to have, enhances the product
Won't-Have: Explicitly out of scope for now
```

For each feature:

```
Feature: Task CRUD
Priority: Must-Have

Description:
> Users can create, view, update, and delete tasks.
> Each task has a title, description, status, and assignee.

Acceptance Criteria (Given/When/Then):
> Given a user is logged in
> When they create a task with title "Fix bug"
> Then the task appears in their task list

> Given a task exists
> When the assignee marks it complete
> Then the status changes to "done"
```

---

## Phase 4: Dependencies

```
What external services or libraries does this need?
───────────────────────────────────────────────────
```

**Example:**
> - Slack API for bot integration
> - SendGrid for email notifications
> - PostgreSQL for data storage

Claude captures:
```
┌────────────────────────────────────────────────────────────┐
│ Dependencies                                               │
├──────────────────┬────────────┬────────────────────────────┤
│ Name             │ Version    │ Purpose                    │
├──────────────────┼────────────┼────────────────────────────┤
│ slack-sdk        │ ^3.0       │ Slack API integration      │
│ sendgrid         │ ^6.0       │ Email notifications        │
│ asyncpg          │ ^0.28      │ PostgreSQL async driver    │
└──────────────────┴────────────┴────────────────────────────┘
```

---

## Phase 5-6: Optional Sections

```
Would you like to add any optional sections?
────────────────────────────────────────────
1. Motivation & Use Cases
2. Competitive Analysis
3. Non-Functional Requirements (performance, security)
4. Skip optional sections
```

**Example NFRs:**
```
Non-Functional Requirements
───────────────────────────

Performance:
> API response time < 200ms for 95th percentile
> Support 100 concurrent users

Security:
> OAuth 2.0 for Slack authentication
> All data encrypted at rest
> No PII in logs
```

---

## Phase 7: Interactive Review

```
Here's your PRD summary:
────────────────────────

Project: Slack Task Manager
Tech: Python 3.12 + FastAPI + PostgreSQL

Must-Have Features:
  - Task CRUD
  - Slack bot integration
  - User authentication

Should-Have Features:
  - Team dashboard
  - Email notifications

Dependencies:
  - slack-sdk ^3.0
  - sendgrid ^6.0
  - asyncpg ^0.28

Would you like to revise anything?
```

You can:
- Edit specific sections
- Add more features
- Change priorities
- Update tech stack

---

## Phase 8: Output

```
Generating PRD files...

Created:
  docs/prd/slack-task-manager/
  ├── index.md           ✓
  ├── what-next.md       ✓
  └── features/
      ├── task-crud.md   ✓
      ├── slack-bot.md   ✓
      └── auth.md        ✓

Your PRD is complete!

Next steps:
  claude /breakdown docs/prd/slack-task-manager/index.md
```

---

## Output Files

### index.md (Main PRD)

```xml
<prd>
  <meta>
    <name>Slack Task Manager</name>
    <slug>slack-task-manager</slug>
    <status>complete</status>
    <created>2025-01-11</created>
  </meta>

  <overview>
    <problem-statement>
      Teams struggle to track tasks across multiple tools...
    </problem-statement>
    <target-users>
      Small development teams of 5-15 people...
    </target-users>
    <value-proposition>
      Simple task tracking without leaving Slack...
    </value-proposition>
  </overview>

  <tech-stack>
    <project-type>greenfield</project-type>
    <technologies>
      <technology name="Python" version="3.12" />
      <technology name="FastAPI" version="0.109" />
      <technology name="PostgreSQL" version="16" />
    </technologies>
  </tech-stack>

  <features>
    <feature slug="task-crud" priority="must-have">
      Task CRUD Operations
    </feature>
    <feature slug="slack-bot" priority="must-have">
      Slack Bot Integration
    </feature>
    <!-- ... -->
  </features>

  <dependencies>
    <dependency name="slack-sdk" version="^3.0">
      Slack API integration
    </dependency>
    <!-- ... -->
  </dependencies>
</prd>
```

### features/task-crud.md

```xml
<feature>
  <meta>
    <name>Task CRUD Operations</name>
    <slug>task-crud</slug>
    <priority>must-have</priority>
    <status>defined</status>
  </meta>

  <description>
    Users can create, view, update, and delete tasks.
    Each task has a title, description, status, and assignee.
  </description>

  <acceptance-criteria>
    <criterion id="AC1">
      <given>A user is logged in</given>
      <when>They create a task with title "Fix bug"</when>
      <then>The task appears in their task list</then>
    </criterion>
    <criterion id="AC2">
      <given>A task exists</given>
      <when>The assignee marks it complete</when>
      <then>The status changes to "done"</then>
    </criterion>
  </acceptance-criteria>
</feature>
```

---

## Tips for Great PRDs

1. **Be specific** - "Fast" is vague; "<200ms response time" is specific
2. **Think about edge cases** - What happens when things go wrong?
3. **Define acceptance criteria** - Given/When/Then makes testing clear
4. **Prioritize ruthlessly** - Must-have should be minimal viable
5. **Consider dependencies** - What external services are needed?

---

## Resuming a PRD

If you need to stop and continue later:

```bash
claude /prd --resume
```

You'll see:
```
Found incomplete PRDs:
  1. slack-task-manager (Phase 5)
  2. analytics-dashboard (Phase 3)

Which would you like to resume?
```

Progress is saved in `what-next.md`.

---

## Next Steps

- [Breaking down your PRD](first-breakdown.md)
- [Features Reference](../skills/prd.md)
- [Back to Quick Start](README.md)
