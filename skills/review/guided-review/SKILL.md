---
name: guided-review
optimized for: Claude Code, Codex
description: Walk a pull request with the user, one file at a time. Use when the user asks to "go through", "walk through", or "review step by step" a PR, a branch, or a set of commits, or invokes /guided-review. Not for an unattended review.
---

# guided-review — Walk a pull request with the user, one file at a time

## Overview

The user reads every change. You are the guide. You group the changes, show
one file and its hunks, explain each hunk, and give one short assessment with
references. The user decides after each file. The user is the reviewer; you
are the reader that never skips a hunk.

**Core principle:** one file per turn, every hunk shown, every assessment
backed by something you looked up.

Write every response in ASD-STE100 Simplified Technical English.
**REQUIRED SUB-SKILL:** use technical-english (Claude Code: invoke
`/technical-english`; Codex: read its `SKILL.md`). If it is not
installed, apply these rules inline: maximum 20 words for each sentence,
active voice, one meaning for each word, no idioms.

## Phase 1 — Load the PR

1. Read the PR: `gh pr view <n> --json title,body,baseRefName,headRefName,commits`.
2. Save the diff: `gh pr diff <n> > <scratchpad>/pr.diff`. Save the file list:
   `gh pr diff <n> --name-only`.
3. Read what the repo documents as its standards (`AGENTS.md`, `CONTRIBUTING.md`, ADRs). Those
   are the references your assessments cite.

Warning: do not use `git diff master...HEAD` for the hunks. The local base
branch is often stale and shows edited files as new files. The saved
`pr.diff` is the only source for hunks. Extract one file with:

```bash
awk '/^diff --git/{p=($0 ~ /<path-fragment>/)} p' pr.diff
```

## Phase 2 — Present the groups

Group the changes by **effect**, not by directory. The commits and the PR
body give the groups. A file can carry hunks from two groups; put the file
under the group that owns most of it and name the other hunk when you show
it.

Show, in this order:

1. One line: PR number, file count, `+added / −removed`, commit count, group
   count.
2. A table with the columns `#`, `Effect`, `Commit`, `Files`, `Size`.
3. Open items the PR body names. Check each one against the repo before you
   list it; a stale note is a finding.
4. One line: "Next: tell me which group to open first." Recommend one group
   and give the reason (smallest group with runtime effect, or the group the
   others depend on).

Stop. Wait for the user.

## Phase 3 — Walk the files

One file per turn. The turn has this shape:

1. Heading: `Group N, file i of M: <path>. <K> hunks.` If the file is a
   contract test with many mechanical hunks, say so.
2. For each hunk, in file order:
   - A bold label: the section or symbol and the line number.
   - The hunk as a `diff` code block. Show the hunk verbatim; shorten only
     long runs of unchanged context.
   - What the hunk does, in one or two sentences.
   - If the hunk belongs to another group, say which commit owns it.
3. **Checks.** Before you assess, look things up: an import that the hunk
   uses, an existing call that the hunk copies, the ADR line that permits or
   forbids the change, a test that locks the string. Cite each as
   `path:line` or a standards document. Also check what the diff cannot show: line
   width, a numbering sequence, a comment that contradicts the code.
4. One line: `My assessment: <verdict>.` Then the findings, if any. Mark
   each finding as `optional` (style) or as a defect. Do not hide a defect
   to keep the turn short.
5. One line: `Next:` with the choices. Always offer "ok" for the next file.
   Offer a one-word command for each proposed fix ("fix", "wrap"). Offer
   "later" to put the finding into the review document.

Never show two files in one turn. Never skip a hunk. Never assess without
a check.

## Quick fixes

When the user picks a fix:

1. Make the edit.
2. Fold it into the commit that owns the file, not into a new commit:
   ```bash
   git commit -q --fixup=<owning-sha> -- <file>
   GIT_SEQUENCE_EDITOR=true git rebase -q -i --autosquash <owning-sha>~1
   ```
3. Report the new SHA of the owning commit. The rebase renumbers every
   later commit, so read `git log --oneline` again before the next fix.
4. Do not push. The user pushes with `--force-with-lease` after the review.

The saved `pr.diff` stays valid; only the SHAs change.

## Deferred findings

When the user says "later", append the finding to
the review document. Ask the user for the path the first time; default to `.scratch/pr-<n>-review.md`, gitignored.
One line per finding: `- [ ] <path>:<line> — <finding> (group N)`. Say the
path once, when you create the file.

## Group and PR end

At the end of a group: `Group N complete: M of M files reviewed.` Then one
line with what was confirmed and what was fixed. Propose the next group.

At the end of the PR:

1. Findings per group, fixed and deferred.
2. Commits that changed SHA.
3. The path of the review document, if one exists.
4. The push command: `git push --force-with-lease`.

## Rationalizations

| Excuse | Reality |
|---|---|
| "The hunk is trivial, I can summarize it" | The user asked to see the diff. Show it. |
| "I can show two small files at once" | One file per turn. Two files double the working set. |
| "The change looks right" | Look it up. An assessment without a check is a guess. |
| "The local diff is fine" | The local base is stale. Use `pr.diff`. |
| "A new commit for the fix is cleaner" | The fix belongs to the commit that made the mistake. Use `--fixup`. |
