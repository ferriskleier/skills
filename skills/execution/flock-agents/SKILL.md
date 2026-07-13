---
name: flock-agents
optimized for: Claude Code, OpenAI Codex
dependencies: mattpocock/skills
description: Implement a whole backlog of tickets overnight, unattended. Give it the spec or parent issue whose build tickets you want built (or a local backlog folder); it pulls the linked tickets from the issue tracker, reads the spec and completed tickets for context, orders the open tickets by dependency, then runs one agent per ticket — each using /implement in its own git worktree, on a model matched to the ticket's difficulty, at high reasoning effort — and leaves you a stack of reviewed branches to merge in the morning.
---

# flock-agents - Run through a backlog autonomously

Implement a whole backlog of tickets overnight, unattended. Give it the spec
(or parent issue) whose build tickets you want built; it pulls the linked
tickets from the configured issue tracker, reads the spec and already-completed
tickets for context, orders the open tickets by dependency, then runs one agent
per ticket — each using the `/implement` skill in its own git worktree, on a
model matched to the ticket's difficulty (see the model matrix below) at high
reasoning effort — and leaves you a stack of reviewed branches to merge in the
morning.

Use when you have a pile of well-formed, session-sized **build tickets** and want
them built AFK. These are the tracer-bullet tickets `/to-tickets` produced
from a spec — not wayfinder's **decision tickets**. So on a wayfinder-originated
effort, point flock at the spec `/to-spec` produced from the map, never the
map itself: the map's children are HITL decisions, and any ticket that still
needs a decision belongs in a grilling or prototype session first, not here.

## Input

Ask for this first if it wasn't given:

- **SOURCE** — the spec or parent issue whose build tickets you'll implement, as
  an issue number or URL. On a local-markdown tracker, SOURCE is instead the
  `.scratch/<feature>/issues/` backlog folder.

How to turn SOURCE into its ticket set is tracker-specific: read
`docs/agents/issue-tracker.md` for how *this* repo lists a spec's child / linked
issues, reads their bodies, and exposes blocking edges. Run
`/setup-matt-pocock-skills` if that file is missing.

## Steps

1. **Gather context.** Read the SOURCE spec / parent issue and the
   already-completed (closed) tickets linked to it. Learn the established
   patterns and conventions, and note what is already done — so agents match
   existing work and never redo a closed ticket.

2. **Enumerate the work.** Pull the open tickets linked to SOURCE from the
   tracker — its child / sub-issues or issues referencing it, filtered to the
   `ready-for-agent` label (per `docs/agents/issue-tracker.md`); on a local
   tracker, the open ticket files in the backlog folder. Skip anything closed,
   merged, or still flagged as needing a decision. For each, capture its
   acceptance criteria and the files it is likely to touch.

3. **Order into phases.** Group tickets by dependency — use the tracker's
   native blocking edges as the primary signal (a ticket is takeable only once
   its blockers are closed), plus file overlap. Tickets that share files or
   depend on another's output must not run in the same parallel batch.
   Independent tickets run together; dependents run in later phases.

4. **Decide how models are chosen — ask the user before launching.** Put the
   choice to the user and wait for an answer:

   - **Automatic (recommended)** — you, the orchestrator, pick a model tier per
     ticket from the matrix below, judging each by its acceptance criteria and
     likely blast radius (captured in step 2).
   - **User-specified** — the user names the model(s): one model for every
     ticket, or a per-tier / per-ticket mapping. Honour it exactly for the
     implementation agents and skip the matrix.

   **Matrix (automatic path).** Match capability tiers, not specific model
   names — this works on whatever runtime you use (Claude Code, OpenAI Codex,
   etc.); map each tier to the best-fitting model your runtime offers. Use the
   smartest, most capable model available for the hardest tickets, and drop to
   faster, cheaper tiers as the work gets more mechanical. When in doubt between
   two tiers, pick the stronger one; a failed overnight run costs more than the
   token difference.

   | Ticket profile | Model tier | Signals |
   | --- | --- | --- |
   | Hardest: UI/UX work, cross-cutting or architectural changes, ambiguous specs, many files touched | Smartest / most capable model available | "redesign", "refactor across", visual/layout acceptance criteria, touches shared frameworks or 5+ files |
   | Normal implementation ticket (the default) | Balanced mid-tier model | one feature or fix, clear acceptance criteria, a handful of files, established patterns to copy |
   | Trivial: formatting, renames, config/doc tweaks, mechanical one-file changes | Fast / cheap model | no design decisions, diff is predictable from the ticket text alone |

   Verifier agents check a diff against acceptance criteria — catching a
   subtle miss is worth the strongest judgment, so run them on the smartest /
   most capable model available regardless of the implementation model, unless
   the user's specification says otherwise.

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

7. **Report.** Post a run summary as a comment on the SOURCE issue (or to a run
   log; on a local tracker, a file in the backlog folder): each ticket's status
   (done / failed / needs-decision), its branch or PR, and anything a human must
   resolve. Leave branches for morning review — do not auto-merge.

## Per-agent instructions

Give each implementation agent exactly this shape:

- You own ONE ticket: `<ticket>` — fetch it from the tracker and read it fully.
- Reference (read-only) the spec and these completed tickets for
  conventions: `<pointers>`. Match existing patterns; do not redo done work.
- Invoke the `/implement` skill on this ticket only: use TDD where possible
  at pre-agreed seams, type-check and run single test files regularly, one
  full test sweep at the end.
- Then run code review, and commit to your worktree branch.
- If you hit a blocker or a real decision, STOP and write why — do not guess.

## Before you run (unattended safety)

- **Pre-allowlist** the shell commands the agents need (test runner,
  typecheck, git, package manager, and the tracker CLI — e.g. `gh` — for
  reading tickets and posting status). Unattended agents auto-approve file
  edits but will stall on a non-allowlisted command with no one to confirm.
- Dynamic workflows must be enabled (on by default for Max/Team/Enterprise;
  Pro users enable them in the Dynamic workflows row of `/config`).
- Worktrees from non-interactive runs are not auto-cleaned; expect to
  `git worktree remove` after merging.
- Fan-out multiplies token usage and draws down rate limits (workflow caps:
  16 agents concurrent, 1000 per run). Size the batch to fit your plan.
