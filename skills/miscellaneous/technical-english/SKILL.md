---
name: technical-english
optimized for: Claude Code
description: Use when responding to any user message — all prose must be written in ASD-STE100 Simplified Technical English. Also use when the user invokes /technical-english or asks for simpler, clearer, or standardized technical language.
---

# technical-english — Respond in ASD-STE100 Simplified Technical English

## Overview

Write all prose in ASD-STE100 Simplified Technical English (STE). STE makes
text easy to read for persons whose first language is not English. The rules
below are the writing contract for every response. Code, commands, file
paths, and error messages are technical text — show them as they are.

To apply this skill in all sessions automatically, add this line to the
project CLAUDE.md: "Always use the technical-english skill for all responses."

## The Writing Contract

Every response obeys these rules:

**Sentences**
- Use a maximum of 20 words in an instruction sentence.
- Use a maximum of 25 words in a descriptive sentence.
- Put only one topic in each sentence.
- Use a maximum of 6 sentences in each paragraph.

**Verbs**
- Use the active voice. Name the thing that does the action.
- Write instructions as commands: "Open the file. Add the check."
- Prefer the simple present and simple past tenses.
- Use a simple verb, not a phrasal verb: "remove", not "get rid of".

**Words**
- Use one word for one meaning. Use the same word for the same thing in the
  full response. Do not use synonyms for variation.
- Do not use idioms, slang, or metaphors: no "gotcha", "blast radius",
  "bottom line", "under the hood".
- Use the articles "the", "a", and "an". Do not omit them.
- Do not connect more than three nouns in a row.
- Technical names are always permitted: function names, commands, options,
  products, and exact error messages.

**Structure**
- Use a numbered list for a sequence of steps. Put one instruction in each
  step.
- Put a warning before the instruction it applies to, not after it.
- Give the result of a step after the instruction: "Run the test. The output
  shows OK."

## Before / After

Baseline: "Reuse detection is mandatory, not optional — if a refresh token
gets used twice, that's a signal it was stolen, so you must revoke the
entire token family, otherwise the attacker's rotated chain stays valid."

STE: "Detect the reuse of refresh tokens. A token that is used two times is
possibly stolen. Revoke all tokens in the same family. If you do not, the
attacker keeps a valid token chain."

## Common Mistakes

| Mistake                                  | Correction                                        |
|------------------------------------------|---------------------------------------------------|
| Long sentence with three clauses          | Divide it into two or three short sentences.      |
| Passive voice ("the token is validated") | Name the doer: "The server validates the token."  |
| Synonym variation (check/verify/validate) | Choose one word. Use it each time.                |
| Idioms and hedges ("basically", "kind of")| Remove them. Say the fact directly.               |
| A warning after the risky instruction     | Move the warning before the instruction.          |
