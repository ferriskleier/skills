---
name: ten-principles
optimized for: Claude Code, Codex, OpenCode
description: Use when writing or changing any code with the ten principles for good software loaded from CLAUDE.md, when the user invokes /ten-principles to apply them to the coding work of this session, or when the user invokes /ten-principles review to review a diff against them. Not for architecture, module boundaries, or design patterns.
---

# ten-principles — Ten principles for good software

## Overview

Ten principles, inspired by Dieter Rams, define what good code looks like.
They are a contract for the style of the code: names, comments, structure,
tests, and size. They are not a contract for architecture. The repo owns
its architecture in ADRs and architectural principles. Follow those first.
Do not use these principles to propose a new module, layer, or pattern.

**Core principle:** less, but better.

Write every response in ASD-STE100 Simplified Technical English.
**REQUIRED SUB-SKILL:** use technical-english for the writing rules. If it is
not installed, apply these rules inline: maximum 20 words for each sentence,
active voice, imperative instructions, one meaning for each word, no idioms.

## The three modes

| Mode          | Activation                                   | Behavior                                                   |
|---------------|----------------------------------------------|------------------------------------------------------------|
| Automatic     | Loaded from CLAUDE.md                        | The principles guide all code you write. No report.        |
| Session       | `/ten-principles`                            | Same as automatic, for the rest of this session.           |
| Review        | `/ten-principles review [base]`              | Review a diff, report the findings, edit after approval.   |

In the automatic and session modes, produce no report and no self-check
output. In the session mode, confirm the activation in one sentence, then
continue with the task of the user.

## The ten principles

### 1. Good code is innovative
It breaks from the default only when the default fails. The break is deliberate and declares itself at the point it happens. New concepts earn their place by removing manual work or complexity, not by being new. Picasso, not ignorance.

### 2. Good code is useful
Every line serves a real use case, an edge case, or a test. If new code does not serve a story, it's invalid. Speculative abstraction for a future that never arrives is avoided.

### 3. Good code is aesthetic
Someone reading the code should be pleased by the structure. It poses no effort in finding implementations or definitions. A simpler structure and clear naming reduce ambiguous implications.

### 4. Good code is self-explanatory
It is understood from looking at it. Functions and variables are named for their actual usage. Comments are only allowed where the solution is not the obvious one. Then the comment names what is surprising and why.

### 5. Good code is unobtrusive
Names let the reader predict the other file without opening it. Patterns are searchable by name. No cleverness draws attention to itself. The code sits quietly in the shape of the code around it.

### 6. Good code is honest
It does not pretend to handle something it doesn't. Naming should make no false promises. Users can rely on looking at the code and expect a behavior without hidden mechanics. Incompleteness or guarantees are made explicit.

### 7. Good code is long-lasting
The code should read just as simple in the future. It must explain durable facts without relying on human or agentic minds and preserve knowledge while allowing code to change. It just feels timeless.

### 8. Good code is thorough down to the last detail
When implementation becomes cheap, a decision is the limiting factor, and verification is the floor. The code shows the chosen form and its edge cases. Tests confirm behavior and contract; they do not discover them.

### 9. Good code is efficient
A reader, human or LLM, needs minimal context to make a change. There is no stale code and no duplicate path. Runtime performance is a different dimension and is not measured here.

### 10. Good code is as little code as possible
Less, but better. Remove until nothing is left that you can remove without breaking the system. Every change removes at least as much complexity as it adds. Simple as possible but not simpler.

## Review mode

### Scope

Read the scope from the invocation:

| Invocation                        | Diff to review                                     |
|-----------------------------------|----------------------------------------------------|
| `/ten-principles review`          | `git diff HEAD` plus untracked files               |
| `/ten-principles review <base>`   | `git diff <base>` — a branch, a tag, or a commit   |

Read the full content of each changed file, not only the hunks. A name or
a duplicate path is only visible in context.

### Report

Group the findings by file. Give each finding these parts, in this order:

1. **Location.** `path/to/file.ext:line`.
2. **Principle.** The number and its short name, for example `#4 self-explanatory`.
3. **Finding.** One or two sentences that say what the code does now.
4. **Change.** One sentence that says what the code must do instead.

A file with no finding is not listed. If the diff has no finding, say so in
one sentence.

Report the findings. Then stop. Ask the user which findings to apply. Do not
edit before the user answers.

### Edits

Apply only the findings that the user approved. Make no other change. After
the edits, list each applied finding with its location in one line.

## Boundary

The repo's ADRs and architectural principles have priority over these
principles. If a finding conflicts with a documented decision, drop the
finding. If a finding proposes to add, move, split, or merge a module, drop
the finding.

## Rationalizations

| Excuse                                            | Reality                                                       |
|---------------------------------------------------|---------------------------------------------------------------|
| "The user wants this fixed, I can edit directly"  | Report first. The user selects the findings.                  |
| "This module should be split, that is style"      | That is architecture. Drop the finding.                       |
| "A comment here helps future readers"             | Only if the solution is surprising. Then name why (#4).       |
| "I will keep the old path for safety"             | A duplicate path is stale code (#9). Remove it.               |
| "This helper will be useful later"                | Code that serves no story is invalid (#2).                    |
