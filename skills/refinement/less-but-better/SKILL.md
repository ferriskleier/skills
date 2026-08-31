---
name: less-but-better
optimized for: Claude Code, Codex
description: Use when the user invokes /less-but-better. Adds one standing task to the whole session, on top of the main task, for both implementation work and reviews. Only the user invokes it; it stays active until the session ends.
---

# less-but-better — Remove as much code as possible without breaking the system

## Overview

"Weniger, aber besser" — less, but better. While this skill is active, every
task carries one extra task: remove code. The system must keep working; the
tests decide what "working" means. Good code is not code with nothing left to
add. Good code is code with nothing left to remove.

**Core principle:** every piece of code you touch leaves your hands smaller
than you found it, or you state that nothing was removable.

## During implementation — the removal pass

Do the main task first. When your change works, the turn is NOT done. Run
the removal pass on the touched scope (each function and file you edited):

1. Reread the touched scope as if you wrote none of it.
2. List the removable weight (see the table below).
3. Verify each removal: grep for callers before you delete a symbol; check
   what the tests cover.
4. Apply the removals directly. Do not ask first.
5. Run the tests. All tests must pass.
6. Report the result as its own line: `Removed: <what> (−N lines)`, or
   `Removed: nothing — <one short reason>`.

The pass is bounded by the touched scope. Removable code elsewhere in the
repo gets one line ("also removable: <path> — <what>") and no edit.

## During a file-by-file review

Reviews propose; they do not edit. For each reviewed file, after the
findings, add one required section:

```
Trim:
- <path>:<line> — <what to remove or reduce> → <replacement> (−N lines)
```

Each entry names the removal, the replacement (or "delete"), and the
estimated line delta. When there is nothing: `Trim: nothing found.` Never
omit the section. Keep defects and trims separate — a bug is a finding, not
a trim.

## What counts as removable

| Weight | Action |
|---|---|
| Unused imports, variables, parameters, functions | Delete (grep for callers first) |
| Commented-out code | Delete; version control remembers it |
| Hand-rolled stdlib (padding loops, manual joins, dedupe scans) | Replace with the builtin |
| Duplicated logic | Keep one copy; the rest calls it |
| Wrappers that only forward | Inline them |
| Branch chains that map a value to a value | One lookup table |
| Flags and options with exactly one caller and one value | Remove the option |

## Safety rules

- A removal must not change observable behavior. Tests pass before and after.
- Warning: a symbol with zero callers in this file can have callers elsewhere.
  Grep the repo before you delete any function, class, or export.
- Untested code paths: do not apply the removal silently. Propose it, or add
  the missing test first.
- Do not touch public API shapes, wire formats, or persisted data.
- When a removal and the main change conflict, the main change wins.

## Red flags — you are about to skip the pass

- "The feature works, I'm done" — the removal pass has not run. Not done.
- "Cleanup is out of scope" — this skill puts it in scope, permanently.
- "I only added two lines" — two lines added to a bloated function is the
  exact trigger. Read the whole function.
- "Deleting feels risky" — an unverified deletion is risky. Verify, then delete.
- "I'll note it for later" — later is only for code outside the touched scope.
