# linear-workflow

Agent skill for tracking software planning and implementation work in Linear.

## What it does

- Reads project-specific Linear config from `.linear-workflow/`.
- Runs a project relevance gate before touching Linear.
- Skips Linear and tells the user when a task belongs to another project or shared tooling.
- Searches for duplicate Linear issues before creating new ones.
- Creates or reuses issues for in-scope plans, fixes, features, refactors, and vague ideas.
- Uses the lifecycle `Backlog → Todo → In Progress → Review → Done`.
- Adds progress comments and final review/testing handoff notes.
- Supports Linear MCP first, with `schpet/linear-cli` as fallback.

## Install

```bash
npx skills add srpiroliro/linear-workflow --skill linear-workflow --yes
```

Or from SSH GitHub URL:

```bash
npx skills add git@github.com:srpiroliro/linear-workflow.git --skill linear-workflow --yes
```

## Project config

Create a `.linear-workflow/` folder in each project. See:

```text
references/project-config-template.md
```

Minimal example:

```json
{
  "team": "Example Team",
  "project": "Example Project",
  "projectScope": {
    "description": "Short description of the product/repository this Linear project tracks.",
    "inScope": ["Features, fixes, and plans for this product/repository"],
    "outOfScope": ["Shared agent skills or unrelated tooling"]
  },
  "trackingPolicy": {
    "projectRelevance": "required",
    "onUnrelatedTask": "skip-linear-and-tell-user",
    "onAmbiguousTask": "ask-before-linear"
  },
  "statuses": {
    "backlog": "Backlog",
    "planning": "Todo",
    "active": "In Progress",
    "review": "Review",
    "done": "Done"
  },
  "donePolicy": "Only move to Done after explicit user acceptance.",
  "toolPreference": ["linear-mcp", "schpet-linear-cli"]
}
```
