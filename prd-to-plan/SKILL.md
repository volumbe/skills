---
name: PRD-to-plan
description: Turn a PRD into a multi-phase implementation plan using tracer-bullet vertical slices, saved as a local Markdown file in ./.plans/. Use when user wants to break down a PRD, create an implementation plan, plan phases from a PRD, or mentions "tracer bullets".
---

# PRD to Plan

Break a PRD into a phased implementation plan using vertical slices (tracer bullets). Output is a Markdown file in `./.plans/<feature-name>/plan.md`.

## Process

### 1. Confirm the PRD is in context

The PRD should already be in the conversation. If it isn't, ask the user to paste it or point you to the file.

### 2. Explore the codebase

If you have not already explored the codebase, do so to understand the current architecture, existing patterns, and integration layers.

### 3. Identify durable architectural decisions

Before slicing, identify high-level decisions that are unlikely to change throughout implementation:

- Route structures / URL patterns
- Database schema shape
- Key data models
- Authentication / authorization approach
- Third-party service boundaries

These go in the plan header so every phase can reference them.

### 4. Draft vertical slices

Break the PRD into **tracer bullet** phases. Each phase is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names, function names, or implementation details that are likely to change as later phases are built
- DO include durable decisions: route paths, schema shapes, data model names

### 5. Quiz the user

Present the proposed breakdown as a numbered list. For each phase show:

- **Title**: short descriptive name
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Should any phases be merged or split further?

Iterate until the user approves the breakdown.

### 6. Write the plan file

Create `./.plans/<feature-name>/` if it doesn't exist. Write the plan as `plan.md` inside that directory (e.g. `./.plans/user-onboarding/plan.md`). The PRD and any other related documents live alongside it in the same folder.

Each phase must include these sections:

- **Blocked by** - list of phases that must be completed before this phase can start. Use "None" for phases with no dependencies. This tells agents which phases can run in parallel.
- **User stories** - which PRD user stories this phase covers
- **What to build** - concise end-to-end description of the vertical slice
- **Acceptance criteria** - checkboxes for verifiable outcomes
- **References** - codebase files to follow/modify and external documentation URLs
- **Implementation Notes** - initially blank, filled after implementation with deviations, decisions, and surprises
- **Implementation Footprint** - initially blank, filled after implementation with list of files created/modified

The plan must begin with an agent instructions callout:

> [!IMPORTANT]
> **Instructions for Agents**
>
> This plan is executed phase-by-phase. After completing each phase:
>
> 1. Update **Implementation Notes** - record deviations from the plan, key decisions made during implementation, and anything surprising or non-obvious.
> 2. Update **Implementation Footprint** - list all files created and modified during the phase.
> 3. Check off **acceptance criteria** - mark completed items with `[x]`.
>
> Read the PRD and the relevant **References** before starting each phase. Phases with no blockers can be implemented in parallel.
