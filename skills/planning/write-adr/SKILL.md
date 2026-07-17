---
name: write-adr
optimized for: Claude Code
dependencies: mattpocock/skills
description: Use when a design discussion, grilling session, or prototype has just settled a significant decision and it should be captured as an Architecture Decision Record — or when the user says "write an ADR", "record this decision", or a why-explanation is about to be lost in chat history.
---

# write-adr — Precipitate a decision into an ADR

## Overview

An ADR captures one architecturally significant decision: the context that
forced it, the decision taken, and the consequences accepted. The code records
*what* was built; the ADR records *why* — the one thing the code can never
recover. This skill runs right after concepts have been worked out: it
interrogates the user until the decision is sharp, then writes it down.

**Core principle:** the ADR is the precipitate, not the discussion. Never
transcribe the chat, and never write rationale the user does not actually
hold — a guessed "why" is worse than none, because future readers will trust it.

## Step 1 — Find the house rules

Discover how this repo already does ADRs before writing anything:

- Look for an existing corpus: `docs/adr/`, `adr/`, `docs/decisions/`,
  `docs/architecture/decisions/`.
- Read its README/template and two or three recent ADRs: ID and filename
  scheme, status vocabulary, extra required fields, index or lint tooling
  (regenerate/run those after writing).
- **House conventions always win over this skill's defaults.**
- No corpus at all? Propose bootstrapping `docs/adr/` with the default format
  below plus a five-line README stating the conventions, so the next ADR has
  something to follow.

## Step 2 — Gather what is already known

Collect facts before questions: re-read the discussion or notes that led here,
the relevant code, and any existing ADR the new decision touches or
contradicts (it may need superseding). Anything you can look up, look up — the
interview is for decisions, not facts.

## Step 3 — Grill

Run a `/grill-me` interview about the decision (from `mattpocock/skills` —
the dependency declared above). If it is not installed, apply its rules
inline: one question at a time, offer your recommended answer with each, wait
for the reply before the next, look facts up in the repo instead of asking.
Interview until every section below can be written without guessing:

1. **Scope.** What exactly was decided? If several decisions got tangled
   together, split them — one decision per ADR.
2. **Does it earn an ADR?** All three must hold: hard to reverse, a real
   rejected alternative, binds future work. If one fails, say so and recommend
   a plain doc or code comment instead; write the ADR only if the user
   overrides.
3. **Context.** What forces made this a decision rather than an obvious
   default? What breaks or hurts without it?
4. **Rejected alternatives.** Which options were seriously considered, and why
   did each lose? A passing mention in the chat ("X might also work") is not a
   considered alternative until the user confirms it was weighed and names why
   it lost. If the user cannot name a rejected alternative from their head,
   the rationale is not held first-hand — stop rather than reconstruct it.
5. **Consequences, both signs.** What becomes easier AND what becomes harder
   or is given up. Which existing code, docs, or habits now contradict the
   decision — and is migrating them part of accepting it?
6. **Status and lineage.** Proposed or Accepted? Does it supersede an existing
   ADR? (Mark the old one `Superseded by <file>`; never rewrite a decision
   in place once others have relied on it.)

## Step 4 — Write it

Use the house format from Step 1; only where the repo has none, default to
Nygard — next free number, kebab-case title, one file:

```markdown
# NNNN. <Decision title, stated as the decision>

## Status

Proposed | Accepted | Superseded by <file>

## Context

The forces in play: what made this a decision and not a default.
Neutral, factual, includes the constraints that ruled options out.

## Decision

What we will do, active voice ("We will ..."). Name the rejected
alternatives and why each lost.

## Consequences

What becomes easier. What becomes harder or is given up. Migration or
cleanup work the decision implies. Both signs, honestly.
```

Keep it short — a page is a good ADR, three is a transcript.

## Step 5 — Land it

- Show the draft and iterate until the user signs it off; they own the
  decision, you own the prose.
- Regenerate any index and run any ADR lint the repo has.
- Commit alongside the change that implements the decision if there is one;
  otherwise as its own commit.

## Common mistakes

- **Skipping the grill because "the chat already covered it"** — the chat has
  facts scattered; the interview forces the rejected alternatives and the
  negative consequences into the open. Those two are what baseline ADRs miss.
- **Transcribing the discussion** — distill; deliberation stays in the chat,
  issue, or design doc.
- **Consequences that only list upsides** — every real decision gives
  something up; if you can't name it, you haven't found it yet.
- **Inventing rationale** — plausible-sounding context the user never stated
  poisons the record. Ask, or leave it out.
- **Bundling decisions** — three decisions in one file are three ADRs.
- **Editing an accepted ADR to change its decision** — supersede with a new
  one; the trail of changed minds is the point.
- **Recording what instead of why** — if the ADR just restates the diff,
  it's not an ADR.
