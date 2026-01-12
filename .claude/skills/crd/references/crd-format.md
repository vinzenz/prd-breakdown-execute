# CRD (Change Request Document) Format Specification

CRD documents capture focused change requests for existing codebases. They are designed to be processed by `/breakdown` for task generation.

## File Location

`{project_root}/docs/crd/{slug}.md`

## Document Structure

```xml
<crd>
  <meta>...</meta>
  <context>...</context>
  <change-request>...</change-request>
  <impact-analysis>...</impact-analysis>
  <requirements>...</requirements>
  <acceptance-criteria>...</acceptance-criteria>
</crd>
```

## Sections

### Meta Section

```xml
<meta>
  <name>Add Dark Mode Toggle</name>
  <slug>dark-mode-toggle</slug>
  <type>feature-add</type>
  <created>2026-01-12</created>
  <status>ready</status>
</meta>
```

| Field | Required | Values | Description |
|-------|----------|--------|-------------|
| `name` | Yes | String | Human-readable change name |
| `slug` | Yes | String | URL-safe identifier (lowercase, hyphens) |
| `type` | Yes | `feature-add`, `feature-modify`, `feature-remove`, `refactor` | Change type |
| `created` | Yes | YYYY-MM-DD | Creation date |
| `status` | Yes | `draft`, `ready`, `in-progress`, `complete` | Current status |

### Context Section

```xml
<context>
  <project-ref>PROJECT.md</project-ref>
  <prd-ref>docs/prd/my-project/index.md</prd-ref>
  <related-features>
    <feature-ref id="settings">User Settings (will be modified)</feature-ref>
    <feature-ref id="auth">Authentication (depends on)</feature-ref>
  </related-features>
</context>
```

| Element | Required | Description |
|---------|----------|-------------|
| `project-ref` | Yes | Path to PROJECT.md |
| `prd-ref` | No | Path to PRD if project was created with /prd |
| `related-features` | Yes | List of features affected by this change |
| `feature-ref` | - | Reference to feature in PROJECT.md with relationship note |

### Change Request Section

```xml
<change-request>
  <summary>Add a dark mode toggle to the settings modal that persists user preference</summary>
  <motivation>Users have requested dark mode support for better accessibility and reduced eye strain during evening use</motivation>
</change-request>
```

| Element | Required | Description |
|---------|----------|-------------|
| `summary` | Yes | 1-2 sentence description of the change |
| `motivation` | Yes | Why this change is needed |

### Impact Analysis Section

```xml
<impact-analysis>
  <affected-files>
    <file action="modify">src/components/SettingsModal.tsx</file>
    <file action="modify">src/api/settings.py</file>
    <file action="create">src/hooks/useTheme.ts</file>
    <file action="create">src/styles/themes.css</file>
    <file action="delete">src/styles/deprecated.css</file>
  </affected-files>

  <affected-features>
    <feature id="settings">Add theme toggle to Appearance section</feature>
  </affected-features>

  <affected-apis>
    <api path="/api/settings" change="Add theme field to settings object"/>
  </affected-apis>

  <affected-schemas>
    <!-- Or: none -->
    <schema model="User" change="No schema change - uses existing settings JSONB"/>
  </affected-schemas>

  <breaking-changes>none</breaking-changes>
  <!-- Or: -->
  <!--
  <breaking-changes>
    <change severity="high" mitigation="Version bump to v2">
      API response shape changes for /api/settings
    </change>
  </breaking-changes>
  -->

  <new-dependencies>
    <!-- Or: none -->
    <dependency name="@radix-ui/react-switch" version="^1.0.0" purpose="Accessible toggle component"/>
  </new-dependencies>

  <scope>small</scope>
  <confidence>high</confidence>
</impact-analysis>
```

| Element | Required | Description |
|---------|----------|-------------|
| `affected-files` | Yes | List of files with action (create, modify, delete) |
| `affected-features` | Yes | Features from PROJECT.md that are impacted |
| `affected-apis` | No | API endpoints that change |
| `affected-schemas` | No | Database models that change |
| `breaking-changes` | Yes | "none" or list of breaking changes with severity |
| `new-dependencies` | No | External packages to add |
| `scope` | Yes | `small` (1-3 files), `medium` (4-8), `large` (9+) |
| `confidence` | Yes | `high`, `medium`, `low` - certainty of impact analysis |

### Requirements Section

```xml
<requirements>
  <requirement id="1" priority="must-have">
    Toggle switch component in settings modal under new "Appearance" section
  </requirement>
  <requirement id="2" priority="must-have">
    Theme preference persisted to user settings via existing PUT /api/settings endpoint
  </requirement>
  <requirement id="3" priority="must-have">
    CSS custom properties (variables) for light and dark color schemes
  </requirement>
  <requirement id="4" priority="should-have">
    Respect system preference (prefers-color-scheme) on first load if no saved preference
  </requirement>
  <requirement id="5" priority="could-have">
    Smooth transition animation when switching themes
  </requirement>
</requirements>
```

| Attribute | Required | Values | Description |
|-----------|----------|--------|-------------|
| `id` | Yes | Integer | Unique requirement ID |
| `priority` | Yes | `must-have`, `should-have`, `could-have` | MoSCoW priority |

### Acceptance Criteria Section

```xml
<acceptance-criteria>
  <criterion id="1">
    <given>User is on the settings page</given>
    <when>User toggles the dark mode switch to "on"</when>
    <then>The entire UI immediately switches to dark theme colors</then>
  </criterion>
  <criterion id="2">
    <given>User has enabled dark mode</given>
    <when>User refreshes the page or returns later</when>
    <then>Dark mode is still active (persisted)</then>
  </criterion>
  <criterion id="3">
    <given>User is a new visitor with no saved preference</given>
    <when>User's system is set to dark mode preference</when>
    <then>App defaults to dark mode</then>
  </criterion>
  <criterion id="4">
    <given>User has saved a theme preference</given>
    <when>User's system preference changes</when>
    <then>App maintains user's explicit choice, ignoring system</then>
  </criterion>
</acceptance-criteria>
```

## Complete Example

```xml
<crd>
  <meta>
    <name>Add Dark Mode Toggle</name>
    <slug>dark-mode-toggle</slug>
    <type>feature-add</type>
    <created>2026-01-12</created>
    <status>ready</status>
  </meta>

  <context>
    <project-ref>PROJECT.md</project-ref>
    <related-features>
      <feature-ref id="settings">User Settings (will be modified)</feature-ref>
    </related-features>
  </context>

  <change-request>
    <summary>Add a dark mode toggle to the settings modal that persists user preference</summary>
    <motivation>Users have requested dark mode support for better accessibility</motivation>
  </change-request>

  <impact-analysis>
    <affected-files>
      <file action="modify">src/components/SettingsModal.tsx</file>
      <file action="modify">src/api/settings.py</file>
      <file action="create">src/hooks/useTheme.ts</file>
      <file action="modify">src/styles/globals.css</file>
    </affected-files>
    <affected-features>
      <feature id="settings">Add theme toggle component</feature>
    </affected-features>
    <breaking-changes>none</breaking-changes>
    <scope>small</scope>
    <confidence>high</confidence>
  </impact-analysis>

  <requirements>
    <requirement id="1" priority="must-have">
      Toggle switch in settings modal under "Appearance" section
    </requirement>
    <requirement id="2" priority="must-have">
      Theme preference saved to user settings via existing API
    </requirement>
    <requirement id="3" priority="must-have">
      CSS variables for light/dark themes
    </requirement>
    <requirement id="4" priority="should-have">
      Respect system preference on first load
    </requirement>
  </requirements>

  <acceptance-criteria>
    <criterion id="1">
      <given>User is on settings page</given>
      <when>User toggles dark mode switch</when>
      <then>UI immediately switches to dark theme</then>
    </criterion>
    <criterion id="2">
      <given>User has dark mode enabled</given>
      <when>User refreshes page or returns later</when>
      <then>Dark mode is still active</then>
    </criterion>
  </acceptance-criteria>
</crd>
```

## Compatibility with /breakdown

CRD format is designed to be processable by `/breakdown`:

| CRD Section | Maps to /breakdown |
|-------------|-------------------|
| `<requirements>` | Feature requirements for task generation |
| `<acceptance-criteria>` | Test requirements for tasks |
| `<impact-analysis>` | Layer planning (which files/features affected) |
| `<context>` | Project context for task generation |
| `<affected-files>` | File scope for tasks |

When running `/breakdown` on a CRD:
```bash
/breakdown docs/crd/dark-mode-toggle.md --project-path /path/to/project
```

The breakdown skill:
1. Detects CRD format (vs PRD format)
2. Reads PROJECT.md for full context
3. Uses `<impact-analysis>` to scope task generation
4. Generates fewer layers (typically 2-3 vs 5 for PRD)
5. Tasks reference existing code from PROJECT.md context

## Status Transitions

```
draft → ready → in-progress → complete
              ↘ abandoned
```

| Status | Meaning |
|--------|---------|
| `draft` | Still being edited, not ready for implementation |
| `ready` | Approved for implementation, can run /breakdown |
| `in-progress` | Tasks generated and being executed |
| `complete` | All tasks finished, PROJECT.md updated |
| `abandoned` | Cancelled, not implemented |
