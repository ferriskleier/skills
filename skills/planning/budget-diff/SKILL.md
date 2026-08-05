---
name: budget-diff
optimized for: Claude Code, Codex, OpenCode
description: Use when the user invokes /budget-diff, states a line budget for a change ("keep it under 150 lines"), or asks for a minimal diff / smallest possible change — before writing any implementation code. Not for ordinary tasks where diff size was never raised.
---

# budget-diff — Set a line budget before coding, then live inside it

## Overview

The budget is set the way a manager sets one for a worker: before the work
starts, and not by the person who will spend it. The executor never grants
themself more room — every extra line is bought from the user, in advance.

**Core principle:** simplicity is the spec. The right solution is the one
that fits the budget and reads plainly stupid at a whiteboard.

**The budget bounds how the task is built, never what the task is.** Every
deliverable the user named gets priced into the budget as they meant it — a
requested test file means test code, a requested flag means the flag.
Shrinking or redefining a deliverable to shrink the number is cheating the
metric. A scope cut is an option you present, priced in lines, for the user
to choose — never a decision you make.

## The two modes

- **No budget given** → *Propose mode*: analyze, propose a budget, and
  **stop until the user approves it**. Not one line of code before approval.
- **Budget given** (a number in the invocation like `/budget-diff 200 <task>`,
  or stated anywhere in the prompt) → *Execute mode*: the user already acted
  as manager; start working inside that number as soon as the scope below is
  settled.

## Budget scope — commit or branch?

A budget bounds a diff, and there are two diffs it could bound. Establish
which one before any code — **ask the user unless they already said**:

- **Commit scope** — only this task's work. Base: the commit `HEAD` points
  at when work starts — record its SHA. Committing mid-task never resets the
  meter; the base stays pinned.
- **Branch scope** — the whole branch/PR diff. Base: the merge-base with the
  default branch (`git merge-base <default-branch> HEAD`). Everything the
  branch already added is budget already spent — measure and report the
  starting spend before writing a line.

In propose mode, ask together with the proposal. In execute mode, it is the
one question allowed before work starts.

## What counts as a line

| Change | Counts against budget? |
|---|---|
| Added implementation line | yes |
| Added test line | yes — tracked as its own split |
| Deleted line | free (deletion is encouraged) |
| Docs, lockfiles, generated files | excluded |

Added lines in the diff, nothing else. Deleting code elsewhere earns no
credit — the metric is additions, not net.

The count is never a mental tally — it is read from the diff itself:

```bash
git diff --numstat "$BASE"   # BASE from the scope above: the pinned SHA or the merge-base
```

Sum the first column (added lines), skip excluded files, split impl vs test
by path. Binary files show `-` and don't count. A tally kept in your head
that disagrees with this command is simply wrong — the diff is the ledger.

## Propose mode

1. **Explore first, thoroughly.** Read the code the task touches. Hunt for
   existing helpers, patterns, and seams to reuse — every reused line is a
   line off the budget. The proposal must be the true minimum, not a guess.
2. **Sketch the minimal design** — the plainly-stupid version. No error
   handling, config, or abstraction the task didn't ask for.
3. **Estimate per component, lean, no padding.** Surprises are handled by
   the breach protocol later, never by a pre-baked margin.
4. **Present and stop:**

   ```
   Proposed budget: 180 lines
     parser        ~60
     runner        ~70
     tests         ~50
   Scope: commit only, or the whole branch? (branch has 42 lines already added)
   ```

   Under branch scope, the lines the branch already added come off the top —
   measure them and show them in the proposal.

   Wait for explicit approval. The user may cut it — that's the point.

## Execute mode

Work in logical steps. After each step, **measure** — run `git diff
--numstat "$BASE"` and sum it — then report the measured count and one
whiteboard sentence describing the **whole system built so far** (not the
last step — this is the drift detector):

```
Budget: 84 impl + 31 test = 115/200 (measured, branch scope)
Whiteboard: parser reads the config, hands a flat dict to the runner.
```

Finish with the same report plus how to try the result.

## Breach protocol

The moment projection shows the remaining work won't fit — **before writing
the overflowing code** — stop and present exactly two options:

1. Simplify the approach to fit the budget (show how).
2. The user explicitly raises the budget.

Never overrun silently. Never write the code first and ask forgiveness —
once it exists, sunk cost argues to keep it.

## Rationalizations — all of them mean stop

| Excuse | Reality |
|---|---|
| "Error handling is basic hygiene" | If the task didn't ask for it, it's budget. Put it in the proposal or leave it out. |
| "Tests need proper scaffolding" | Subprocess e2e + setUp/tearDown for a 20-line tool triples the test bill. Call the function, assert the value. |
| "It's stdlib-only, so it's minimal" | Minimal dependencies ≠ minimal diff. The budget counts lines. |
| "It's only a few lines over" | A few lines over is the breach protocol's call, and the protocol belongs to the user. |
| "A rounder/higher budget gives room" | Padding is the executor granting themself budget. Estimate lean; breaches go to the user. |
| "I'll delete elsewhere to make room" | Deletions are free but earn no credit. Additions are the metric. |
| "A sample data file counts as the test file" | Redefining a deliverable to dodge its lines is a scope cut. Price real test code; if it doesn't fit, that's the breach protocol. |
| "Running it by hand counts as testing" | If the user asked for tests, manual verification doesn't discharge it. Assertions or a priced-out scope proposal. |
| "My running tally says it fits" | The tally isn't the metric — the diff is. Run `git diff --numstat`; when they disagree, the diff wins. |
| "Earlier commits on the branch shouldn't count" | Under branch scope they are the budget's first spend. The scope belongs to the user; report the starting count, don't renegotiate it. |
| "Scope is obvious, no need to ask" | Commit vs branch changes the number by every line the branch already added. One question, asked once, before code. |

## Red flags — stop and re-read this skill

- Code written before a budget exists or is approved
- Code written before the scope (commit vs branch) is settled
- A count reported without running `git diff --numstat`
- A "while I'm here" addition
- A step finished without its budget report
- An estimate produced after the code
- A budget raised by anyone but the user
- A requested deliverable quietly redefined, downgraded, or dropped to fit the number
