# Agent Skills

A collection of the agent skills I like to use most for planning, product thinking, and frontend specification work.

## Planning & Design

- **write-a-prd** — Create a PRD through an interactive interview, codebase exploration, and module design. Persists output to `./.plans/<feature-name>/prd.md`.

```bash
npx skills@latest add volumbe/skills/write-a-prd
```

- **prd-to-plan** — Turn a PRD into a multi-phase implementation plan using tracer-bullet vertical slices. Persists output to `./.plans/<feature-name>/plan.md`.

```bash
npx skills@latest add volumbe/skills/prd-to-plan
```

- **write-ui-spec** — Create a UI specification through frontend exploration, user interview, and structured page/component design. Persists output to `.plans/<feature-name>/ui.md`.

```bash
npx skills@latest add volumbe/skills/write-ui-spec
```

## Credits

- **write-a-prd** and **prd-to-plan** are adapted from [mattpocock/skills](https://github.com/mattpocock/skills).
- The versions in this repo keep the original workflows, with changes to how PRDs and plans are persisted so they land in local project folders instead of GitHub issues or flatter plan outputs.
