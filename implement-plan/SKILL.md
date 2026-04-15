---
name: implement-plan
description: Execute an approved implementation plan from .plans, either by implementing a specific phase or carrying the full plan through completion. Use this whenever the user asks to implement a plan, execute a phase, work through a plan in .plans, or explicitly wants parallel/subagent execution based on phase blockers.
---

# Implement a Plan

Use this skill to turn an existing plan into working code. It is designed for plans stored in `./.plans/<feature-name>/plan.md`, especially plans created by `prd-to-plan`.

This skill covers two common modes:

- Implement a specific phase from a plan
- Implement an entire plan, optionally with parallel work when the user explicitly asks for subagents or parallel execution

## Gather context first

Start by locating the relevant plan file. If the user did not provide the exact path, find the likely match under `./.plans/`.

Read the plan before doing any coding. If adjacent planning docs exist, read them too:

- `./.plans/<feature-name>/prd.md`
- `./.plans/<feature-name>/ui.md`

Also read the codebase files referenced by the target phase before deciding on an implementation approach.

## Choose the execution mode

### Mode 1: Single-phase implementation

Use this when the user asks for a specific phase such as:

- "Implement phase 2"
- "Do the first tracer bullet"
- "Carry out the auth phase from this plan"

Before coding:

1. Confirm which phase is in scope.
2. Check its **Blocked by** section.
3. If prerequisite phases are incomplete, stop and tell the user the phase is blocked unless they want you to implement the blockers first.

### Mode 2: Full-plan implementation

Use this when the user asks for the whole plan to be executed.

Work phase-by-phase in dependency order. Treat **Blocked by** as the source of truth for sequencing.

If the user explicitly asks for subagents, delegation, or parallel execution:

- Parallelize only phases whose blockers are already complete
- Give each worker a single phase or tightly-scoped slice with clear file ownership
- Avoid overlapping write sets between concurrent workers
- Merge and verify each completed phase before marking it done in the plan

If the user does not explicitly ask for subagents or parallel work, implement locally without delegation.

## Always begin with an implementation plan

Before editing code, begin in the agent's plan mode or equivalent planning step.

Create a concrete implementation plan for the requested work. That plan should include:

- The target phase or phases
- The files you expect to modify
- Any new files you expect to create
- Key architectural or integration decisions for this implementation pass
- Validation steps you will run
- For parallel work, which worker owns which phase and file set

Do not skip this step. The purpose is to translate the high-level plan phase into an execution-ready coding plan before touching the codebase.

## Implementation workflow

### 1. Read the phase requirements carefully

For each target phase, extract:

- **Blocked by**
- **User stories**
- **What to build**
- **Acceptance criteria**
- **References**

Use these sections to keep the implementation aligned with the original plan rather than drifting into ad hoc feature work.

Treat **Acceptance criteria** as markdown task checkboxes, not plain bullet points. In other words, they should look like:

- `[ ] Criterion not yet complete`
- `[x] Criterion completed and verified`

### 2. Explore the codebase

Inspect the referenced files and nearby implementation patterns before coding.

Prefer reusing existing abstractions and established conventions. If the phase implies deeper module boundaries or reusable seams, implement them that way rather than scattering shallow logic.

### 3. Implement the phase end-to-end

For a single phase, complete the full vertical slice described by the phase.

For a full plan, finish one unblocked phase at a time unless the user explicitly requested parallel work and the dependency graph allows it.

Do not mark a phase complete just because the code compiles. A phase is only complete when its acceptance criteria are actually verified.

### 4. Verify before updating the plan

Run the most relevant validation for the work you just completed. This can include:

- Targeted tests
- Full test suites
- Linting
- Typechecking
- Build verification
- Manual product checks when appropriate

If verification fails, fix the issue before marking the phase complete. If you cannot finish verification, say so explicitly and do not check off the relevant acceptance criteria.

## Update the original plan file

After a phase is implemented and verified, update the original `./.plans/.../plan.md`.

This is not optional. The plan is the shared project record, and it should reflect reality after implementation.

For each completed phase:

1. Check off completed **Acceptance criteria** items with `[x]`
2. Fill in **Implementation Notes**
3. Fill in **Implementation Footprint**

Do not rewrite acceptance criteria as plain bullets. Preserve the checkbox format and only flip items from `[ ]` to `[x]` when they are genuinely complete and verified.

### What goes in Implementation Notes

Record the real implementation story, including:

- Deviations from the original phase description
- Important technical decisions made during implementation
- Surprising edge cases or integration issues
- Follow-up work that became visible during execution

Keep this concise but useful to the next engineer or agent.

### What goes in Implementation Footprint

List the files you created and modified for that phase.

Use a simple flat list such as:

- `app/routes/settings.tsx`
- `components/settings-form.tsx`
- `lib/settings/validators.ts`

When implementing the full plan, update the plan after each verified phase rather than waiting until the very end. This keeps the plan accurate even if the work is interrupted.

## Reporting back to the user

When you finish, summarize:

- Which phase or phases were implemented
- What verification you ran
- Any acceptance criteria that remain incomplete
- Whether the original plan file was updated
- Any blockers, follow-ups, or risks still remaining

If you only completed part of the requested work, be explicit about what remains and why.

## Example prompts this skill should handle well

- "Implement phase 1 from `./.plans/new-onboarding/plan.md`"
- "Carry out the billing plan in `.plans/billing-rewrite/plan.md`"
- "Implement the whole plan and use subagents wherever phases can run in parallel"
- "Do phase 3, then update the plan with notes and file footprints"
