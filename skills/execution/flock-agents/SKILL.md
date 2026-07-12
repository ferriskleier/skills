---
name: flock-agents
optimized for: Claude Code
dependencies: mattpocock/skills
description: Implement a whole backlog of tickets overnight, unattended. Point it at a backlog folder; it reads the epic and completed stories for context, orders the open tickets by dependency, then runs one agent per ticket — each using /implement in its own git worktree, on a model matched to the ticket's difficulty, at high reasoning effort — and leaves you a stack of reviewed branches to merge in the morning.
---

# flock-agents - Run through a backlog autonomiously

Implement a whole backlog of tickets overnight, unattended. Point it at a
backlog folder; it reads the epic and already-completed stories for context,
orders the open tickets by dependency, then runs one agent per ticket — each
using the `/implement` skill in its own git worktree, on a model matched to
the ticket's difficulty (see the model matrix below) at high reasoning
effort — and leaves you a stack of reviewed branches to merge in the morning.

Use when you have a pile of well-formed, session-sized tickets and want them
built AFK. Do NOT use for planning or fog: tickets must already be scoped
(e.g. from `to-tickets`). Any ticket that still needs a
decision belongs in a grilling or prototype session first, not here.

## Input

- BACKLOG: path to the folder of tickets/stories to work through.
  Ask for it if it wasn't given.

## Steps

1. **Gather context.** Read the epic/spec and the already-completed stories
   in or near BACKLOG. Learn the established patterns and conventions, and
   note what is already done — so agents match existing work and never redo
   a closed ticket.

2. **Enumerate the work.** List the open tickets in BACKLOG. Skip anything
   closed, merged, or flagged as needing a decision. For each, capture its
   acceptance criteria and the files it is likely to touch.

3. **Order into phases.** Group tickets by dependency. Tickets that share
   files or depend on another's output must not run in the same parallel
   batch. Independent tickets run together; dependents run in later phases.

4. **Pick a model per ticket** using the matrix below. Judge each ticket by
   its acceptance criteria and likely blast radius (captured in step 2) —
   when in doubt between two tiers, pick the stronger one; a failed overnight
   run costs more than the token difference.

   | Ticket profile | Model | Signals |
   | --- | --- | --- |
   | Hardest: UI/UX work, cross-cutting or architectural changes, ambiguous specs, many files touched | `claude-fable-5` | "redesign", "refactor across", visual/layout acceptance criteria, touches shared frameworks or 5+ files |
   | Normal implementation ticket (the default) | `claude-opus-4-8` | one feature or fix, clear acceptance criteria, a handful of files, established patterns to copy |
   | Trivial: formatting, renames, config/doc tweaks, mechanical one-file changes | `claude-sonnet-4-6` | no design decisions, diff is predictable from the ticket text alone |

   Verifier agents check a diff against acceptance criteria — that is a
   normal-tier task; run them on `claude-opus-4-8` regardless of the
   implementation model.

5. **Launch the flock** as a background dynamic workflow, so it survives
   overnight and resumes if interrupted. For each ticket spawn one agent:
   - in its own git worktree (isolated branch — no collisions),
   - on the model chosen in step 4 at **high** reasoning effort,
   - with the per-agent instructions below.
   Run phases in order; within a phase, run agents in parallel.

6. **Verify each result.** After an agent commits, a separate verifier agent
   checks the diff against that ticket's acceptance criteria. On failure,
   return the ticket to its agent ONCE with the failure notes; if it still
   fails, stop and flag it. Never loop unbounded overnight.

7. **Report.** Write a run summary to BACKLOG (or a run log): each ticket's
   status (done / failed / needs-decision), its branch or PR, and anything a
   human must resolve. Leave branches for morning review — do not auto-merge.

## Per-agent instructions

Give each implementation agent exactly this shape:

- You own ONE ticket: `<ticket>`. Read it fully.
- Reference (read-only) the epic and these completed stories for
  conventions: `<pointers>`. Match existing patterns; do not redo done work.
- Invoke the `/implement` skill on this ticket only: use TDD where possible
  at pre-agreed seams, type-check and run single test files regularly, one
  full test sweep at the end.
- Then run code review, and commit to your worktree branch.
- If you hit a blocker or a real decision, STOP and write why — do not guess.

## Before you run (unattended safety)

- **Pre-allowlist** the shell commands the agents need (test runner,
  typecheck, git, package manager). Unattended agents auto-approve file
  edits but will stall on a non-allowlisted command with no one to confirm.
- Dynamic workflows must be enabled (on by default for Max/Team/Enterprise;
  Pro users enable them in the Dynamic workflows row of `/config`).
- Worktrees from non-interactive runs are not auto-cleaned; expect to
  `git worktree remove` after merging.
- Fan-out multiplies token usage and draws down rate limits (workflow caps:
  16 agents concurrent, 1000 per run). Size the batch to fit your plan.
