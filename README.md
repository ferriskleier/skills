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
- **[use-codex](./skills/execution/use-codex/SKILL.md)** — Delegate a task to the OpenAI Codex CLI (GPT-5.x) as an autonomous subagent.
  - Use when you want to offload a coding/research/review task, get a second-opinion model, or fan out parallel work to Codex instead of (or alongside) Claude subagents.
  - Shells out to `codex exec` with a self-contained prompt, runs it unattended, and reads back the final message — with recipes for structured output, read-only review, and flocking over git worktrees.

### Review

_Nothing here yet._

### Refinement

_Nothing here yet._

### Productivity

_Nothing here yet._

### Miscellaneous

_Nothing here yet._
