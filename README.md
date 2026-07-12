# Ferris' LLM Skills

A collection of skills for coding agents that I made and use myself.

## Installation

```bash
npx skills@latest add ferriskleier/skills
```

Then pick the skills and agents you want when prompted.

## When to use which

### Planning

_Nothing here yet._

### Execution

- **[flock-agents](./skills/execution/flock-agents/SKILL.md)** — Implement a whole backlog of tickets overnight, unattended.
  - Use when you have a folder of well-scoped, session-sized tickets and want them built AFK — not for planning or tickets that still need decisions.
  - Reads the epic and completed stories for context, orders open tickets by dependency, then runs one agent per ticket in its own git worktree, on a model matched to the ticket's difficulty.
  - Verifies each result against the ticket's acceptance criteria and leaves you a stack of reviewed branches to merge in the morning — never auto-merges.

### Review

_Nothing here yet._

### Refinement

_Nothing here yet._

### Productivity

_Nothing here yet._

### Miscellaneous

_Nothing here yet._
