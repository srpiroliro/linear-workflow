# Project Config Template

Create this folder in each repository that uses `linear-workflow`:

```text
.linear-workflow/
  config.json
  workflow.md
```

## `.linear-workflow/config.json`

```json
{
  "team": "Example Team",
  "project": "Example Project",
  "statuses": {
    "backlog": "Backlog",
    "planning": "Todo",
    "active": "In Progress",
    "review": "Review",
    "done": "Done"
  },
  "statusPolicy": {
    "backlog": "Use for ideas or requests that need discovery, research, clarification, or planning before a concrete plan exists.",
    "planning": "Use for planning-only requests once a concrete plan exists.",
    "active": "Use when implementation starts.",
    "review": "Use when implementation is complete and ready for user validation.",
    "done": "Use only after explicit user acceptance."
  },
  "donePolicy": "Only move to Done after explicit user acceptance.",
  "toolPreference": ["linear-mcp", "schpet-linear-cli"]
}
```

## `.linear-workflow/workflow.md`

```md
# Linear Workflow

Use Linear for all planning and implementation work.

Lifecycle:

Backlog → Todo → In Progress → Review → Done

- Backlog: vague idea, discovery/research/clarification needed.
- Todo: concrete plan exists, implementation not started.
- In Progress: implementation active.
- Review: implementation complete, awaiting user validation.
- Done: user explicitly accepted or asked to close.

Final review comments must include:
- what changed
- files/areas touched
- tests run
- manual test steps
- risks/follow-ups
```
