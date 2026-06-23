# Linear Workflow Config Template

Create this folder in each repository, product workspace, shared skill repo, or workstream that uses `linear-workflow`:

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
  "workScope": {
    "description": "Short description of the product, repository, workspace, or workstream this Linear config tracks.",
    "inScope": [
      "Features, fixes, refactors, migrations, and plans for this configured work scope"
    ],
    "outOfScope": [
      "Shared agent skills, unrelated tooling, or work for a different repository/product/workstream"
    ]
  },
  "trackingPolicy": {
    "scopeRelevance": "required",
    "minimumTaskSize": "standalone-change-or-plan",
    "onUnrelatedTask": "skip-linear-and-tell-user",
    "onSmallTask": "skip-linear-and-tell-user",
    "onAmbiguousTask": "ask-before-linear"
  },
  "taskSize": {
    "trackWhen": [
      "requires a plan",
      "changes product/application behavior",
      "fixes a bug",
      "adds a feature",
      "requires meaningful testing or review"
    ],
    "skipWhen": [
      "commit only",
      "push only",
      "create worktree only",
      "small text edit",
      "formatting only",
      "read-only explanation"
    ]
  },
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

Use Linear for eligible planning and implementation work.

Before touching Linear, confirm the request is inside this config's work scope and is large enough to track as a standalone issue.

Skip Linear for unrelated scopes and tiny operational tasks such as commit-only, push-only, worktree-only, formatting-only, or small text edits.

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
