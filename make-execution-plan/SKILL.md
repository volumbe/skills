---
name: make-execution-plan
description: Create a concrete, file-level execution plan by delegating the planning pass to the Claude CLI. Use when the user asks to use Claude to make an execution plan, wants a detailed implementation checklist, or wants an execution.md under .plans.
---

# Make Execution Plan

Use this skill to turn an existing request, PRD, or plan into a concrete execution checklist by asking the Claude CLI for a second planning pass. The execution plan can stay in memory and be relayed back to the user, or be saved as `./.plans/<feature-name>/execution.md` when the user asks for a file.

When persisted, `execution.md` must be a working checklist for an implementing agent, not a prose memo. Use markdown task checkboxes so progress can be tracked directly in the file.

## Workflow

### 1. Gather local context

Before invoking Claude, collect the context Claude should reason from:

- The user's current implementation request and decisions already made in the thread
- Any referenced `prd.md`, `plan.md`, `ui.md`, screenshots, tickets, or prior checklist
- The relevant repo instructions, especially build/test commands and style constraints
- Likely files and modules involved, discovered with `rg`, `rg --files`, and targeted reads
- Verification commands the implementer should run

If the user mentions a `.plans` folder but not an exact path, find the likely folder under `./.plans/`. Read nearby planning docs in this order when present:

1. `prd.md`
2. `plan.md`
3. `ui.md`
4. Existing `execution.md`

### 2. Decide output mode

Use in-memory mode unless the user explicitly asks to create or update a file.

- **In-memory mode**: relay Claude's execution plan in the chat and follow it for the implementation.
- **File mode**: write Claude's execution plan to `./.plans/<feature-name>/execution.md`.

Do not overwrite an existing `execution.md` without checking whether it should be updated or replaced. Prefer updating it when it clearly belongs to the same feature.

### 3. Invoke Claude CLI

Prefer a bounded, non-editing Claude invocation. Claude should plan only; the current agent remains responsible for edits.

Use a prompt file or heredoc for substantial context so shell quoting does not distort paths or Markdown.

Recommended command shape:

```bash
claude --bare --tools "" --no-session-persistence --max-budget-usd 1 -p "$(cat /tmp/execution-plan-prompt.md)"
```

If repo inspection by Claude is necessary and worth the cost, allow read-only tools explicitly and keep the prompt narrow:

```bash
claude --permission-mode plan --allowedTools "Read,Grep,Glob,Bash(rg *),Bash(find *)" --no-session-persistence -p "$(cat /tmp/execution-plan-prompt.md)"
```

If a Claude run hangs or produces no output after about 90 seconds, stop it and rerun with `--bare --tools ""` using tighter context gathered by this agent.

### 4. Claude prompt template

Include enough context for Claude to be specific. The prompt should tell Claude not to edit files and to return only the checklist.

```text
Create a concrete execution plan for the current coding agent to follow.

Do not edit files. Return only the detailed execution checklist and verification commands.
Be file-level and sequencing-level, not architectural fluff.

Repository: <absolute repo path>
User request: <what is being implemented>

Locked decisions and constraints:
- <decision>
- <decision>

Repo instructions:
- <package manager / test commands / style constraints>

Planning references:
- <path to prd.md and summary, if present>
- <path to plan.md and summary, if present>
- <path to ui.md and summary, if present>

Likely files/modules:
- <path>: <why it matters>
- <path>: <why it matters>

Required output shape:
1. Execution phases in order, each using markdown task checkboxes
2. Each phase contains `Checklist`, `Execution Notes`, and `Follow-Ups` sections
3. Exact files likely to create or modify
4. Data/API/UI wiring details as actionable checklist items
5. Edge cases and access-control rules as checklist items
6. Verification commands as checklist items
7. Manual smoke checks as checklist items

Keep it implementation-ready. Avoid product rationale.

The persisted execution plan must include this instruction near the top:

> [!IMPORTANT]
> This is a working execution checklist. As implementation progresses, mark items with `[x]` only after the code is changed and the relevant verification has passed. Leave blocked or unverified work unchecked and add a short note explaining why.
```

### 5. Review and use the plan

After Claude returns:

1. Check the plan against the user's locked decisions.
2. Remove or correct steps that conflict with repo instructions or discovered code patterns.
3. Relay the final plan in memory, or write it to `execution.md` in file mode.
4. If the user wants implementation next, follow the resulting checklist directly.

When writing `execution.md`, include a short header with:

- Feature name
- Source references used
- Date created
- Note that Claude CLI generated the first draft and the current agent reviewed it
- The working-checklist instruction block shown above

Then structure the body with unchecked markdown tasks:

```md
## Phase 1: <name>

### Checklist

- [ ] Change `<file>` to ...
- [ ] Add/update tests for ...
- [ ] Run `<verification command>`
- [ ] Manually verify ...

### Execution Notes

<!-- Fill this in after implementation. Include deviations from the plan, key decisions, surprising behavior, and verification results. -->

### Follow-Ups

<!-- Fill this in after implementation. Include deferred work, remaining risks, blocked items, and cleanup that should happen later. -->
```

Do not mark any checklist item as complete when creating the execution plan. Completion belongs to the agent doing the implementation.
Do not pre-fill `Execution Notes` or `Follow-Ups` with speculative content. Use short placeholder comments so the implementing agent knows what to add after executing the phase.

## Example triggers

- "Use Claude CLI to make an execution plan."
- "Make a more detailed implementation checklist."
- "Create `.plans/foo/execution.md` from this PRD and plan."
- "I don't like your plan; ask Claude for a concrete execution plan."
