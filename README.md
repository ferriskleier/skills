# Ferris' LLM Skills

A collection of skills for coding agents that I made and use myself.

## Installation

```bash
npx skills@latest add ferriskleier/skills
```

Then pick the skills and agents you want when prompted.

## When to use which

### Planning

- **[budget-diff](./skills/planning/budget-diff/SKILL.md)** — Set a line budget before coding, then live inside it.
  - Use when a change must stay small ("keep it under 150 lines") or when you want a minimal-diff proposal approved before any code is written.
  - First settles whether the budget bounds the current commit only or the whole branch/PR diff, then measures spend from `git diff --numstat` against that base — the diff is the ledger, never a self-kept tally.
  - Proposes a lean per-component budget and stops for approval when none was given; projected overruns go back to the user as simplify-or-raise, never silent breaches.
- **[write-adr](./skills/planning/write-adr/SKILL.md)** — Precipitate a just-settled decision into an Architecture Decision Record.
  - Use right after a design discussion, grilling session, or prototype has settled a significant decision — before the "why" gets lost in chat history.
  - Discovers the repo's existing ADR conventions (or bootstraps them), then runs a `/grill-me` interview (from [mattpocock/skills](https://github.com/mattpocock/skills)) until the decision, its rejected alternatives, and both-signed consequences can be written without guessing.
  - Writes one Nygard-format ADR per decision and refuses to invent rationale the user doesn't actually hold.

### Execution

- **[flock-agents](./skills/execution/flock-agents/SKILL.md)** — Implement a whole backlog of tickets overnight, unattended.
  - Use when you have a folder of well-scoped, session-sized tickets and want them built AFK — not for planning or tickets that still need decisions.
  - Reads the epic and completed stories for context, orders open tickets by dependency, then runs one agent per ticket in its own git worktree, on a model matched to the ticket's difficulty.
  - Verifies each result against the ticket's acceptance criteria and leaves you a stack of reviewed branches to merge in the morning — never auto-merges.
- **[use-codex](./skills/execution/use-codex/SKILL.md)** — Delegate a task to the OpenAI Codex CLI (GPT-5.x) as an autonomous subagent.
  - Use when you want to offload a coding/research/review task, get a second-opinion model, or fan out parallel work to Codex instead of (or alongside) Claude subagents.
  - Shells out to `codex exec` with a self-contained prompt, runs it unattended, and reads back the final message — with recipes for structured output, read-only review, and flocking over git worktrees.

### Review

- **[guided-review](./skills/review/guided-review/SKILL.md)** — Walk a pull request with the user, one file at a time.
  - Use when you want to read a PR yourself and have the agent guide you through it — not for an unattended review.
  - Groups the changes by effect into one table, then shows one file per turn: every hunk as a diff, what it does, and one assessment backed by a look-up (`path:line`, a standards doc, an ADR).
  - After each file you pick: continue, apply a one-word quick fix folded into the owning commit with `--fixup`, or defer the finding into a review document.
  - Responds in ASD-STE100 Simplified Technical English (see [technical-english](./skills/miscellaneous/technical-english/SKILL.md)).

### Refinement

_Nothing here yet._

### Productivity

- **[let-me-write-it](./skills/productivity/let-me-write-it/SKILL.md)** — Guide the user to write the code themselves; never write it for them.
  - Use when you want to type every character yourself and have the agent act as a navigator — pointing at files, explaining causes, and giving instructions instead of code.
  - Every answer follows a fixed shape — location, cause, instruction, check — and never contains code blocks, diffs, or line-for-line pseudo-code; identifiers and keywords are named sparingly in prose.
  - Responds in ASD-STE100 Simplified Technical English (see [technical-english](./skills/miscellaneous/technical-english/SKILL.md)) and stays active until you explicitly end the mode.

### Miscellaneous

- **[technical-english](./skills/miscellaneous/technical-english/SKILL.md)** — Respond in ASD-STE100 Simplified Technical English.
  - Use when you want every response in short, clear, standardized English — invoke with /technical-english, or add it to your CLAUDE.md so it applies to every request automatically.
  - Enforces the STE writing contract: sentences of at most 20–25 words, active voice, imperative instructions, one meaning per word, no idioms, warnings before the step they apply to.
  - Code, commands, file paths, and error messages stay verbatim — only the prose is simplified.
