# Agent Skills

A small collection of agent skills.

## Planning

- **make-execution-plan** — Create a concrete, file-level execution plan by delegating the planning pass to the Claude CLI. Relays the plan in memory or persists it as `.plans/<feature-name>/execution.md`.

```bash
npx skills@latest add volumbe/skills/make-execution-plan
```
