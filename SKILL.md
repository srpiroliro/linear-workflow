---
name: linear-workflow
description: Track software planning and implementation work in Linear. Use before planning or implementing fixes, features, refactors, behavior changes, or vague ideas that need discovery. Handles project config from .linear-workflow/, duplicate checks, issue creation/reuse, status transitions, progress comments, and final review handoff.
---

# Linear Workflow

Use this skill before planning or implementing software work that should be represented in Linear.

This skill is reusable across projects. Keep project-specific Linear settings in the repository, not in this skill.

## Trigger

Use this skill whenever the user asks to:

- implement a fix, feature, refactor, migration, or behavior change;
- investigate and fix a bug;
- plan how to fix or build something;
- continue, revise, or complete previously planned implementation work;
- capture an idea that needs discovery, research, clarification, or planning before implementation.

Do not create a new issue for small read-only explanations, searches, or reviews unless the work becomes a plan or implementation task.

## Project Config

Read project config from:

1. `.linear-workflow/config.json` preferred
2. `.linear-workflow/config.md` fallback
3. `AGENTS.md` fallback if project config is missing

If no config exists, ask the user for the Linear team, project, and status names before creating issues.

Recommended `.linear-workflow/config.json`:

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

## Tool Preference

### Context-preserving execution

When the `subagent` tool is available to the parent agent, delegate Linear bookkeeping to a fresh-context subagent instead of performing it in the main conversation. This keeps duplicate searches, issue payload drafting, API/CLI output, and status/comment operations out of the parent context.

Use a narrow subagent task that includes only:

- the user request summary;
- the repo Linear config (`team`, `project`, statuses, done policy);
- the intended workflow action: search/reuse/create/update/comment/status transition;
- the required title prefix rule: new issues start with `🤖 `;
- the expected compact result shape: issue identifier, URL, status, whether it was created/reused/updated, and any blocker.

Subagent execution rules:

- Prefer `context: "fresh"` so the child does not inherit the full parent conversation.
- Use the `delegate` agent or another available Linear-capable agent. If a project provides a specialized Linear agent, prefer it.
- For initial issue search/create/reuse, wait for the subagent result before starting implementation so the parent can reference the issue.
- For non-blocking progress comments, async subagents are acceptable if the parent does not need the result immediately.
- The child must not make product, scope, architecture, or implementation decisions; it only performs Linear bookkeeping from the parent-provided summary.
- If the child cannot access Linear MCP/CLI/auth, it must report the blocker. The parent then falls back to inline Linear tooling.
- Do not use `pi -p` or spawn a separate Pi CLI process for this workflow.

Example subagent shape:

```typescript
subagent({
  agent: "delegate",
  context: "fresh",
  task: "Perform only Linear bookkeeping for this request. Read .linear-workflow/config.json if needed. Search for duplicates, create or reuse the issue, apply the configured status, and return only: identifier, URL, status, action taken, and blockers. Do not modify project files."
})
```

### Linear tool preference inside the parent or child

Use whichever Linear integration is available and authenticated:

1. Prefer Linear MCP for common operations because it returns structured data and avoids shell escaping.
2. Use `schpet/linear-cli` when MCP is unavailable, unauthenticated, or lacks a needed operation.
3. Use `schpet/linear-cli` raw GraphQL/API as an advanced fallback.

If using `schpet/linear-cli`, ensure the skill is installed:

```bash
npx skills add schpet/linear-cli --skill linear-cli --yes
```

If the `linear` binary is unavailable, prefix commands with:

```bash
npx @schpet/linear-cli
```

For markdown descriptions and comments, prefer files over inline shell arguments:

- `--description-file` for issue descriptions
- `--body-file` for comments

## Required Workflow

### 1. Load config

Read `.linear-workflow/config.json` and extract:

- `team`
- `project`
- `statuses.backlog`
- `statuses.planning`
- `statuses.active`
- `statuses.review`
- `statuses.done`
- `donePolicy`

Validate that required status names exist if the tool can list statuses. If a configured status is missing, pause and tell the user what to create.

### 2. Search for duplicates

Before creating an issue, search existing issues in the configured team/project using concise keywords from the user request.

Linear MCP pattern:

- `linear_list_issues` with `team`, `project`, `query`, `includeArchived: false`.

CLI pattern:

```bash
npx @schpet/linear-cli issue query \
  --team "<team>" \
  --project "<project>" \
  --search "<keywords>" \
  --all-states \
  --json
```

Reuse an issue when it is clearly the same request or a direct continuation. Add a short comment explaining the new context instead of creating a duplicate.

### 3. Choose initial status

Use the configured lifecycle:

```text
Backlog → Todo → In Progress → Review → Done
```

Choose status by request type:

- `backlog`: vague idea, needs discovery/research/clarification before a safe plan can be made.
- `planning`: user asks to plan and a concrete plan can be produced now.
- `active`: user asks to implement now or you are starting implementation.
- `review`: implementation is complete and ready for user validation.
- `done`: only after explicit user acceptance or close request.

### 4. Create or update issue

Issue title must be short and understandable.

For every new issue created by this skill, prefix the title with the robot emoji and a space: `🤖 `. This applies to all creation paths, including Linear MCP and `schpet/linear-cli`. Do not add a second robot prefix if the proposed title already starts with `🤖 `. When searching for duplicates, use meaningful keywords from the title/request without relying on the emoji. When reusing an existing issue, do not rename it solely to add the robot prefix unless the user explicitly asks.

Issue description must include:

- user request summary;
- intended scope;
- relevant repo context or constraints;
- current status rationale;
- for planning requests, the plan or a path/link to the plan.

If reusing an issue, update status if needed and add a comment with the new request context.

### 5. Add progress comments

Add a concise Linear comment when moving between meaningful stages, especially:

- planning created;
- implementation started;
- significant milestone completed;
- blocker discovered;
- implementation ready for review.

### 6. Final review handoff

When implementation is complete:

1. Move the issue to `statuses.review`.
2. Add a final comment containing:
   - what changed;
   - files or areas touched;
   - commands/tests run and their results;
   - manual test steps for the user;
   - known risks, limitations, or follow-ups.

Do not move to `statuses.done` unless the user explicitly accepts the work or asks to close the issue.

## CLI Examples

Create a planning issue:

```bash
npx @schpet/linear-cli issue create \
  --team "<team>" \
  --project "<project>" \
  --state "<planning status>" \
  --title "🤖 <clear title>" \
  --description-file /tmp/linear-description.md
```

Create an implementation issue:

```bash
npx @schpet/linear-cli issue create \
  --team "<team>" \
  --project "<project>" \
  --state "<active status>" \
  --title "🤖 <clear title>" \
  --description-file /tmp/linear-description.md
```

Add a comment:

```bash
npx @schpet/linear-cli issue comment add <ISSUE_ID> --body-file /tmp/linear-comment.md
```

Update status:

```bash
npx @schpet/linear-cli issue update <ISSUE_ID> --state "<status>"
```

## Failure Handling

If Linear authentication is unavailable:

1. If the Linear operation was delegated to a subagent, have the child report the blocker compactly and return control to the parent.
2. Try the other configured tool path: MCP ↔ CLI.
3. If the subagent path failed due tool/auth availability, retry inline in the parent with MCP/CLI before declaring tracking blocked.
4. If neither MCP nor CLI works, pause and tell the user Linear tracking is blocked.
5. Do not silently continue with implementation when tracking is required by project rules.

If duplicate search fails due tooling/auth but Linear creation works, search with a different tool or narrower query before creating a new issue.

Never use `pi -p` or spawn a separate Pi CLI process as a fallback for this skill. Use subagents when available, otherwise use inline Linear MCP/CLI.
