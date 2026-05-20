---
name: make-execution-plan
description: Create a concrete, file-level execution plan by delegating the planning pass to the Claude CLI. Use when the user asks to use Claude to make an execution plan, wants a detailed implementation checklist, or wants an execution.md under .plans.
---

# Make Execution Plan

Use this skill to turn an existing request, PRD, or plan into a concrete execution plan by asking the Claude CLI for a second planning pass. The execution plan can stay in memory and be relayed back to the user, or be saved as `./.plans/<feature-name>/execution.md` when the user asks for a file.

When persisted, `execution.md` must be a working execution document for an implementing agent, not a prose memo. Use markdown task checkboxes so progress can be tracked directly in the file, and allow concrete implementation notes, pseudo-code, real code snippets, query sketches, schema sketches, or command examples when they help the implementer move faster.

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

Prefer a non-editing Claude invocation that gives Claude enough room to inspect the
repo and think through the plan. Claude should plan only; the current agent remains
responsible for edits.

Use a prompt file or heredoc for substantial context so shell quoting does not distort paths or Markdown.

Recommended command shape:

```bash
claude --model opus --effort max --permission-mode bypassPermissions --no-session-persistence -p "$(cat /tmp/execution-plan-prompt.md)"
```

When the plan depends on codebase details, let Claude use its normal planning-mode
tool access. Do not suppress tools or cap the run with a max budget unless the user
explicitly asks for a constrained/cheap planning pass.

Use `bypassPermissions` only in a trusted local workspace. In non-interactive
`-p` runs, this avoids Claude parking while waiting for tool permission prompts and
allows it to inspect the repository before returning the plan.

If Claude must be prevented from using tools for a specific reason, say so in the
prompt and use a tool-restricted command deliberately:

```bash
claude --model opus --effort max --bare --no-session-persistence -p "$(cat /tmp/execution-plan-prompt.md)"
```

Planning runs may take several minutes before producing output. Do not assume Claude
is hanging just because there is no output after 90 seconds. Before interrupting,
check whether the process is still active and give it more time unless the user has
asked for a quick answer or the process is clearly stuck.

### 4. Claude prompt template

Include enough context for Claude to be specific. The prompt should tell Claude not to edit files and to return only the execution plan.

```text
Create a concrete execution plan for the current coding agent to follow.

Do not edit files. Return only the detailed execution plan and verification commands.
Be file-level and sequencing-level, not architectural fluff.
Use markdown checkboxes for trackable work, but include pseudo-code, real code snippets, SQL/query sketches, schemas, or examples where they make the implementation clearer.

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
2. Each phase contains `Checklist`, optional `Implementation Details`, `Execution Notes`, and `Follow-Ups` sections
3. Exact files likely to create or modify
4. Data/API/UI wiring details as actionable checklist items
5. Edge cases and access-control rules as checklist items or implementation details
6. Verification commands as checklist items
7. Manual smoke checks as checklist items
8. Pseudo-code, SQL, TypeScript snippets, schema sketches, or examples when useful

Keep it implementation-ready. Avoid product rationale.

The persisted execution plan must include this instruction near the top:

> [!IMPORTANT]
> This is a working execution plan. As implementation progresses, mark checklist items with `[x]` only after the code is changed and the relevant verification has passed. Leave blocked or unverified work unchecked and add a short note explaining why. Code snippets and pseudo-code are guidance, not completion markers.
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
- The working execution plan instruction block shown above

Then structure the body with unchecked markdown tasks and optional implementation details:

```md
## Phase 1: <name>

### Checklist

- [ ] Change `<file>` to ...
- [ ] Add/update tests for ...
- [ ] Run `<verification command>`
- [ ] Manually verify ...

### Implementation Details

Optional concrete notes, pseudo-code, TypeScript snippets, SQL/query sketches, route shapes, or examples that make the checklist easier to execute.

```ts
// Example shape only; adapt to the actual code.
```

### Execution Notes

<!-- Fill this in after implementation. Include deviations from the plan, key decisions, surprising behavior, and verification results. -->

### Follow-Ups

<!-- Fill this in after implementation. Include deferred work, remaining risks, blocked items, and cleanup that should happen later. -->
```

Do not mark any checklist item as complete when creating the execution plan. Completion belongs to the agent doing the implementation.
Do not pre-fill `Execution Notes` or `Follow-Ups` with speculative content. Use short placeholder comments so the implementing agent knows what to add after executing the phase.
Do not force every detail into a checkbox. Use prose and snippets for details that explain how to implement a checked item.

## Example triggers

- "Use Claude CLI to make an execution plan."
- "Make a more detailed implementation checklist."
- "Create `.plans/foo/execution.md` from this PRD and plan."
- "I don't like your plan; ask Claude for a concrete execution plan."
