---
name: review-and-improve
optimized for: Claude Code, Codex
disable-model-invocation: true
description: Final review of the current branch against the repo's ADRs and existing code. Fixes every finding and repeats unattended until no SEVERE or MODERATE finding is left. All fixes stay uncommitted.
---

# review-and-improve — Review the branch, fix the findings, repeat until clean

## Overview

You are on the branch where the work was implemented. This is its final
review. The user invokes the skill and leaves; they come back to a branch
that was reviewed several times, with every fix applied in the working tree.
The run is unattended: you decide, you proceed, you never wait for an answer.

The run is a loop of **rounds**. Each round is: review, overview, fix. The
loop ends when a review finds no SEVERE and no MODERATE finding.

**Core principle:** ask of every change "how did we already implement this
in similar places, or how do we set the rules for this?". Where the codebase
answers, the branch follows the answer. Where it does not, the new concept
is built to look as if it always existed here.

All changes stay in the working tree. The user commits after the run.

## Review

Every round reviews the whole branch: the diff against the base branch,
uncommitted changes included, so the fixes of earlier rounds are reviewed
too.

### 1. Rules

Search the repo for what it wrote down about how to build: ADRs, architecture
documents, `AGENTS.md` / `CLAUDE.md`, contribution guides. Read every one
that bears on the touched code.

### 2. Precedents

This step runs in full whether step 1 found rules or not.

For each thing the branch adds or changes, search the code outside the diff
for the places that already do the same kind of thing: the **precedents**.
Read them completely, with their wiring, naming, error handling, and tests.

Done when each concept the branch introduces has one of:

- a rule or a precedent it must follow;
- `new here`: you searched, and the codebase has no rule and no earlier
  case. A new concept is welcome. Judge it against the general conventions
  of the codebase: layering, naming, error handling, test style.

A written rule wins over a precedent.

### 3. Findings

Read every hunk of the diff with these two questions:

- **Does it fit the codebase?** A change that invents its own way where a
  rule or precedent exists is a finding; the fix is the existing way. A
  `new here` concept that looks foreign next to its neighbors is a finding
  too.
- **Would a careful reviewer accept it?** Bugs, unhandled cases and errors,
  security, missing or weak tests, dead code, and what the change forgot:
  callers, migrations, configuration, documentation.

Each finding has a location, the problem, the evidence (a rule, a precedent
as `path:line`, or the input that breaks it), and the fix.

| Severity | Meaning |
|---|---|
| SEVERE | Wrong behavior, data loss, a security hole, a failing check, or a violation of an ADR. |
| MODERATE | A departure from a clear precedent, a realistic case left unhandled, new behavior without a test, or a structure that makes the next change expensive. |
| LOW | Style, naming, small cleanups; the code is correct and consistent without the fix. |

## Overview

Print the findings of the round, one bullet each, then continue with the
fixes in the same turn:

```
Round N

SEVERE
- <path>:<line> — <finding>

MODERATE
- <path>:<line> — <finding>

LOW
- <path>:<line> — <finding>
```

An empty category prints `- none`.

## Fix

Fix every finding of the round, LOW included, SEVERE first.

- Build each fix the way its rule or precedent is built: same structure,
  same helpers, same naming, same test style.
- Stay inside what the branch set out to do.
- Run the repo's checks (tests, lint, type check) after the fixes. Repair
  what the fixes broke.

A finding whose fix needs a decision only the user can make stays unfixed
and is marked `open`, with the options. A finding that proves wrong is
marked `rejected`, with the reason.

Then start the next round with a new review.

## Loop end

The loop ends when a review returns no SEVERE and no MODERATE finding;
`open` findings do not count. The LOW findings of that last review stay
unfixed and go into the report.

Stop after round 8 even when findings remain, and say so in the first line
of the report.

## Final report

1. One line: the number of rounds and the result.
2. For each round, every finding by severity with the change that fixed it
   and the rule or precedent the fix follows.
3. What remains: the LOW findings of the last review, and every `open`
   finding with its options.
4. The result of the checks and the output of `git status --short`. The
   user reviews the changes and commits.

## Rationalizations

| Excuse | Reality |
|---|---|
| "There are no ADRs, so there is no architecture to check" | The existing code is the architecture. Find the precedents. |
| "My design is cleaner than the existing one" | Consistency wins. Use the existing way. |
| "I should confirm the findings with the user before I fix them" | Nobody is there. Print the overview and fix. |
| "The last round changed little, a new review is not needed" | Every fix is new, unreviewed code. Only a review ends the loop. |
| "A commit per round keeps the history clean" | The user commits. The working tree holds all fixes. |
