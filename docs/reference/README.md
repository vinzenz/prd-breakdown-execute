# Reference

Quick reference for commands, arguments, and file formats.

---

## Reference Documents

| Document | Description |
|----------|-------------|
| [Commands](commands.md) | All commands with arguments |
| [File Formats](file-formats.md) | XML and JSON schemas |

---

## Quick Command Reference

```bash
# PRD Creation
claude /prd
claude /prd --resume

# Breakdown
claude /breakdown <prd-path>
claude /breakdown <prd-path> --output-dir <path>
claude /breakdown <prd-path> --project-path <path>

# Execute
claude /execute <tasks-path>
claude /execute <tasks-path> --resume
claude /execute <tasks-path> --max-parallel 5
claude /execute <tasks-path> --dry-run
```

---

## Key Files

| File | Location | Purpose |
|------|----------|---------|
| PRD | `docs/prd/{slug}/index.md` | Main PRD document |
| Features | `docs/prd/{slug}/features/*.md` | Feature details |
| Analysis | `docs/tasks/{slug}/analysis.json` | PRD analysis |
| Layer Plan | `docs/tasks/{slug}/layer_plan.json` | Task organization |
| Tasks | `docs/tasks/{slug}/{layer}/*.xml` | Task specifications |
| State | `docs/tasks/{slug}/execute-state.json` | Execution state |

---

## Layer Reference

| Layer | Name | Purpose |
|-------|------|---------|
| 0 | Setup | Project initialization (greenfield) |
| 1 | Foundation | Models, migrations, config |
| 2 | Backend | APIs, services, logic |
| 3 | Frontend | UI components, state |
| 4 | Integration | E2E wiring, polish |

---

## Status Values

### Global

| Status | Description |
|--------|-------------|
| `pending` | Not started |
| `in_progress` | Running |
| `completed` | Done |
| `abandoned` | Failed 5 times |

### Task

| Status | Description |
|--------|-------------|
| `pending` | Not started |
| `in_progress` | Implementing |
| `verifying` | Being verified |
| `verified` | Passed verification |
| `completed` | Merged to main |
| `failed` | Will retry |
| `abandoned` | Won't retry |

---

## Next Steps

- [Commands Reference](commands.md)
- [File Formats](file-formats.md)
- [Troubleshooting](../troubleshooting.md)
