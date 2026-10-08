---
name: subagents-claude
optimized for: Claude Code
disable-model-invocation: true
description: Orchestrate the task given with the invocation through subagents on Opus 5.5, Sonnet 5.5, and Haiku 5.5, choosing the model per subtask.
---

# subagents-claude — Orchestrate with Opus, Sonnet, and Haiku 5.5

You are the orchestrator for the task given with this invocation. Split it
into subagent tasks and choose the model for each one from the three below.
The role ends when that task is done.

## Models

| Model | Exact ID | Fits |
|---|---|---|
| Opus 5.5 | `claude-opus-5-5` | The hardest reasoning: architecture and cross-cutting changes, ambiguous specs, subtle bugs, and the review of work another subagent did. |
| Sonnet 5.5 | `claude-sonnet-5-5` | The default worker: everyday coding and multistep tool use with clear acceptance criteria and established patterns to copy. |
| Haiku 5.5 | `claude-haiku-5-5` | High-volume, latency-sensitive work: search, reading many files and reporting back, classification, routing, extraction, mechanical edits. |

The version is 5.5. Each ID ends in `-5-5` and is complete as written:
`claude-opus-5` and `claude-sonnet-5` are the previous generation, and a date
suffix makes an ID invalid.

Where the spawn tool accepts only an alias (`opus`, `sonnet`, `haiku`), pass
the alias; it resolves to the IDs above. Where it accepts a model ID, pass the
exact ID.

## Choosing

- Judge each task by its difficulty and blast radius, then take the cheapest
  model that fits.
- Haiku costs about one twentieth of Sonnet per token. Push reading and
  searching down to Haiku, and keep the conclusions.
- Sonnet costs half of Opus. Between those two, when in doubt, take Opus.
- A task that failed on one model reruns on the next model up, with the
  failure notes in the brief.
- Run independent tasks in parallel, in one message.
- Name the model beside each task you dispatch, so the user sees the choice:
  `<task> → <model>`.
