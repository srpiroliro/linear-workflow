# linear-workflow

Agent skill for tracking software planning and implementation work in Linear.

## What it does

- Bootstraps new repositories by creating `.linear-workflow/` config after user confirmation.
- Reads per-repository Linear config from `.linear-workflow/`.
- Runs scope relevance and task-size gates before touching Linear.
- Skips Linear and tells the user when a task belongs to another scope or is too small/operational to track.
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

## New repository setup

When `.linear-workflow/config.json` is missing, the skill asks whether to configure Linear workflow. The user can choose to:

1. use an existing Linear project;
2. create a new Linear project;
3. track issues at team level with `project: null`;
4. skip Linear workflow for that repository.

The skill should create `.linear-workflow/config.json`, `.linear-workflow/workflow.md`, and optionally a minimal `AGENTS.md` pointer only after confirmation.

## Repository/workspace config

Create a `.linear-workflow/` folder in each repository or workspace that should use this skill. See:

```text
references/project-config-template.md
```

Minimal example:

```json
{
  "team": "Example Team",
  "project": null,
  "workScope": {
    "description": "Short description of the product, repository, workspace, or workstream this Linear config tracks.",
    "inScope": ["Features, fixes, and plans for this configured scope"],
    "outOfScope": ["Shared agent skills, unrelated tooling, or other workstreams"]
  },
  "trackingPolicy": {
    "scopeRelevance": "required",
    "minimumTaskSize": "standalone-change-or-plan",
    "onUnrelatedTask": "skip-linear-and-tell-user",
    "onSmallTask": "skip-linear-and-tell-user",
    "onAmbiguousTask": "ask-before-linear"
  },
  "taskSize": {
    "trackWhen": ["requires a plan", "changes behavior", "fixes a bug", "adds a feature"],
    "skipWhen": ["commit only", "push only", "create worktree only", "small text edit", "formatting only"]
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
