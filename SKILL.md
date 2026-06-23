---
name: linear-workflow
description: Track eligible software planning and implementation work in Linear. Use before planning or implementing fixes, features, refactors, behavior changes, vague ideas that need discovery, or setting up Linear workflow in a new repository. Handles per-repository config from .linear-workflow/, new-repo bootstrap, scope/task-size eligibility gates, duplicate checks, issue creation/reuse, status transitions, progress comments, and final review handoff.
---

# Linear Workflow

Use this skill before planning or implementing software work that may need to be represented in Linear. Loading the skill does not mean Linear must be touched; first run the eligibility gates below.

This skill is reusable across repositories, products, and teams. Keep each repository's Linear configuration and work scope in `.linear-workflow/`, not in this skill.

## Trigger

Use this skill whenever the user asks to:

- implement a fix, feature, refactor, migration, or behavior change;
- investigate and fix a bug;
- plan how to fix or build something;
- continue, revise, or complete previously planned implementation work;
- capture an idea that needs discovery, research, clarification, or planning before implementation.

Do not create a new issue for small read-only explanations, searches, or reviews unless the work becomes an eligible plan or implementation task.

## Repository/Workspace Config

Read config from:

1. `.linear-workflow/config.json` preferred
2. `.linear-workflow/config.md` fallback
3. Legacy `AGENTS.md` fallback only when it contains explicit Linear team/project/status config

If no `.linear-workflow/config.json` exists, run the New Repository Bootstrap below. Do not create/search/update Linear issues for project work until the repository has config or the user explicitly chooses a config for this session.

The skill owns the reusable workflow rules. Repository files should only hold local configuration and, optionally, a short `AGENTS.md` pointer that tells agents to use this skill.

Recommended `.linear-workflow/config.json`:

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
      "requires meaningful testing or review",
      "touches API, database, automation, scheduling, sending, or user-facing UI"
    ],
    "skipWhen": [
      "commit only",
      "push only",
      "create worktree only",
      "small text edit",
      "formatting only",
      "read-only explanation",
      "external/shared skill maintenance unless this config explicitly tracks that skill"
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

Set `project` to `null` when issues should be tracked at team level without a Linear project.

## New Repository Bootstrap

Run this bootstrap when `.linear-workflow/config.json` is missing or clearly incomplete.

### Bootstrap principles

- Do not touch Linear for issue tracking until config exists or the user explicitly selects a session config.
- Do not automatically create a Linear project.
- Do not assume every repository needs its own Linear project.
- The agent may create config files, but only after user confirmation.
- Keep `AGENTS.md` optional and minimal; do not duplicate this skill's workflow rules there.

### Bootstrap flow

1. Inspect the repository enough to suggest a work scope: repo name, README/package metadata, top-level directories, and existing agent instructions.
2. Tell the user no `.linear-workflow/config.json` exists and ask whether to configure Linear workflow for this repository.
3. Offer exactly these setup choices:
   - Use an existing Linear project.
   - Create a new Linear project.
   - Track issues at team level without a Linear project (`project: null`).
   - Skip Linear workflow for this repository.
4. If the user chooses an existing project:
   - list/search Linear teams and projects when tooling is available;
   - ask the user to choose the matching team/project when ambiguous;
   - write `.linear-workflow/config.json` and `.linear-workflow/workflow.md` after confirmation.
5. If the user chooses a new project:
   - ask for the Linear team and project name/description;
   - create the Linear project only after explicit confirmation;
   - if tooling cannot create projects, ask the user to create it manually and then continue config;
   - write `.linear-workflow/config.json` and `.linear-workflow/workflow.md` after the project exists or the user confirms the intended project name.
6. If the user chooses team-level tracking:
   - set `project` to `null`;
   - search/create issues by team only;
   - write `.linear-workflow/config.json` and `.linear-workflow/workflow.md` after confirmation.
7. If the user chooses to skip:
   - do not create config;
   - do not create Linear issues;
   - continue the user's task without Linear tracking.

### Files created during bootstrap

Create these files after confirmation:

```text
.linear-workflow/
  config.json
  workflow.md
```

Optionally create or update `AGENTS.md` only when the user wants persistent agent instructions or the repository already uses `AGENTS.md`. The content should be a short activation pointer, for example:

```md
# Project Agent Rules

## Linear workflow

For eligible planning or implementation work in this repository, use the `linear-workflow` skill. Repository-specific Linear config lives in `.linear-workflow/`.
```

Do not paste duplicate lifecycle, gate, duplicate-search, or handoff rules into `AGENTS.md`; those belong in this skill.

### Bootstrap tracking

Do not create a Linear issue merely to track the bootstrap unless the user explicitly asks. Once config exists, future eligible project work can be tracked normally.

## Tracking Eligibility Gates

Before any Linear search, creation, status update, or comment, run both gates:

1. Scope relevance gate — does this request belong to the configured work scope?
2. Task significance gate — is this request large/meaningful enough to track as a Linear issue?

The skill can be loaded only to decide that Linear should be skipped. Do not treat the configured Linear project as a general agent-work bucket.

Use `.linear-workflow/config.json` fields `project`, `workScope` (or legacy `projectScope`), `trackingPolicy`, and `taskSize` to decide.

### Scope relevance gate

Pass this gate when the request clearly changes, plans, debugs, researches, or implements something for the configured work scope. The scope may be a product, repository, package, shared skill, team workstream, or other explicitly configured area. It is not necessarily a product project.

Fail this gate when the request is about:

- a shared/global agent skill while the current config tracks an application/product;
- an application/product while the current config tracks a shared skill;
- Pi/agent configuration unrelated to the configured work scope;
- another repository, package, service, client, product, or workstream;
- Linear bookkeeping itself, unless the bookkeeping is for an in-scope issue;
- general advice, read-only explanation, or research that does not create an in-scope plan or implementation task.

If this gate fails:

1. Do not query Linear.
2. Do not create or update a Linear issue.
3. Briefly tell the user that the task appears outside the configured work scope and Linear tracking was skipped.
4. Offer to track it only if the user provides the correct Linear config or explicitly says this work belongs to the current scope.

Suggested message:

```text
I’m not using Linear for this because the task appears outside the configured work scope: <scope/project>. If this should be tracked, tell me which Linear config/project to use.
```

If relevance is ambiguous, ask one focused question before touching Linear. Default to not creating/updating Linear until the user confirms the task belongs in the configured scope.

### Task significance gate

Pass this gate when the request is meaningful enough to be a standalone Linear task, such as:

- it requires a plan or discovery;
- it changes product/application behavior;
- it fixes a bug or adds a feature;
- it touches API, database, automation, scheduling, sending, user-facing UI, deployment, or data migrations;
- it requires meaningful testing, review, or rollout notes;
- it is part of an already tracked issue and the Linear update is useful.

Fail this gate for small operational/mechanical tasks, such as:

- commit-only or push-only requests;
- creating/checking out a worktree or branch;
- tiny copy/text edits;
- formatting-only or lint-only changes;
- reading, searching, explaining, or summarizing without a resulting plan/implementation;
- tool installation/update chores that do not affect the configured work scope;
- renames or file moves that are not part of a larger tracked change.

If this gate fails:

1. Do not create a new issue.
2. Do not update Linear unless there is already an active issue and a short comment would add useful context.
3. Briefly tell the user Linear was skipped because the task is too small/operational to track as a standalone issue.

Suggested message:

```text
I’m not creating a Linear issue for this because it is a small operational task, not a standalone feature/fix/plan.
```

## Tool Preference

### Context-preserving execution

When the `subagent` tool is available to the parent agent, delegate Linear bookkeeping to a fresh-context subagent instead of performing it in the main conversation. This keeps duplicate searches, issue payload drafting, API/CLI output, and status/comment operations out of the parent context.

Use a narrow subagent task that includes only:

- the user request summary;
- the repo Linear config (`team`, `project`, work scope, tracking policy, task size policy, statuses, done policy);
- the parent agent's explicit eligibility decision: scope relevance and task significance;
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
  task: "Perform only Linear bookkeeping for this request after the parent has confirmed it passes the scope and task-size gates. Read .linear-workflow/config.json if needed. Search for duplicates, create or reuse the issue, apply the configured status, and return only: identifier, URL, status, action taken, and blockers. Do not modify project files."
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

### 1. Load config, bootstrap if needed, and run eligibility gates

If `.linear-workflow/config.json` is missing or incomplete, run the New Repository Bootstrap before any Linear API/CLI calls for issue tracking. If the user declines setup, stop the Linear workflow for this repository and continue the task without Linear tracking.

When config exists, read `.linear-workflow/config.json` and extract:

- `team`
- `project` (string project name/id, or `null` for team-level tracking)
- `workScope` or legacy `projectScope`
- `trackingPolicy`
- `taskSize`
- `statuses.backlog`
- `statuses.planning`
- `statuses.active`
- `statuses.review`
- `statuses.done`
- `donePolicy`

Run the Tracking Eligibility Gates before any Linear API/CLI call, including status-list validation.

If the request is outside the configured work scope, stop the Linear workflow and tell the user Linear tracking was skipped because the task does not appear to belong to the configured scope. If relevance is ambiguous, ask the user to confirm before touching Linear.

If the request is too small or operational to track as a standalone issue, stop the Linear workflow and tell the user Linear tracking was skipped because the task is below the configured tracking threshold. If an active issue is already known, add a comment only when useful.

Only after both gates pass, validate that required status names exist if the tool can list statuses. If a configured status is missing, pause and tell the user what to create.

### 2. Search for duplicates

Before creating an issue, search existing issues in the configured team and configured project when `project` is not `null`. If `project` is `null`, search by team only. Use concise keywords from the user request.

Linear MCP pattern:

- `linear_list_issues` with `team`, `project`, `query`, `includeArchived: false`.

CLI pattern:

```bash
npx @schpet/linear-cli issue query \
  --team "<team>" \
  --project "<project-if-configured>" \
  --search "<keywords>" \
  --all-states \
  --json
```

Omit `--project` when `project` is `null`:

```bash
npx @schpet/linear-cli issue query \
  --team "<team>" \
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
- `planning`: user asks to plan and a concrete plan can be produced now, or a previously Backlog issue now has a concrete plan and is waiting for user approval.
- `active`: user asks to implement now or you are starting implementation.
- `review`: implementation is complete and ready for user validation.
- `done`: only after explicit user acceptance or close request.

Status freshness rule:

Keep the Linear issue status current throughout the work. At every meaningful phase transition, update the issue status before claiming that phase is complete or moving on to the next phase.

Required transitions:

- `statuses.backlog` → `statuses.planning` once a concrete plan, design, or spec exists and is waiting for user approval.
- `statuses.planning` → `statuses.active` when implementation starts.
- `statuses.active` → `statuses.review` when implementation is complete and ready for user validation.
- `statuses.review` → `statuses.done` only after explicit user acceptance or an explicit close request.

Do not leave issues in stale statuses. If a status update fails, report the blocker and do not imply Linear is up to date. If work becomes blocked without a status transition, keep the current status but add a blocker comment.

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

### 5. Promote plan-ready Backlog issues to planning

When a concrete plan, design, or implementation spec is complete for an issue that started in `statuses.backlog`, move the issue to `statuses.planning` (usually Todo) before asking the user to approve the plan. Do not leave plan-ready work in Backlog while it waits for approval.

This applies whether the plan was written inline, saved to a file, or produced by a planning subagent. Add a concise Linear comment that includes:

- plan summary or plan file path/link;
- that the plan is ready for user approval;
- any known blockers or decisions still needed before implementation.

Do not move the issue to `statuses.active` until implementation actually starts. If the status update or comment fails, report the blocker instead of claiming the planning handoff is complete.

### 6. Add progress comments

Add a concise Linear comment when moving between meaningful stages, especially:

- planning created;
- planning completed and waiting for approval;
- implementation started;
- significant milestone completed;
- blocker discovered;
- implementation ready for review.

### 7. Final review handoff

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

Create a planning issue when a project is configured:

```bash
npx @schpet/linear-cli issue create \
  --team "<team>" \
  --project "<project>" \
  --state "<planning status>" \
  --title "🤖 <clear title>" \
  --description-file /tmp/linear-description.md
```

Create a planning issue for team-level tracking when `project` is `null`:

```bash
npx @schpet/linear-cli issue create \
  --team "<team>" \
  --state "<planning status>" \
  --title "🤖 <clear title>" \
  --description-file /tmp/linear-description.md
```

Create an implementation issue when a project is configured:

```bash
npx @schpet/linear-cli issue create \
  --team "<team>" \
  --project "<project>" \
  --state "<active status>" \
  --title "🤖 <clear title>" \
  --description-file /tmp/linear-description.md
```

Omit `--project` for implementation issues when `project` is `null`.

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
