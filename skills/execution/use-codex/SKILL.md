---
name: use-codex
optimized for: Claude Code
description: Use when you want to delegate work to the OpenAI Codex CLI (GPT-5.x) as an autonomous subagent — offloading a coding/research/review task, getting a second-opinion model, or fanning out parallel work to Codex instead of (or alongside) Claude subagents.
---

# use-codex

## Overview

`codex` is the OpenAI Codex CLI (an agentic coding tool like Claude Code, but GPT-backed). You can shell out to it with `Bash` to hand a whole task to Codex and let it work autonomously, then read back its result. Use this to offload work, get a different model's take, or run a flock of Codex agents in parallel.

**Core principle:** `codex exec` is the non-interactive entrypoint. Give it a self-contained prompt, run it unattended, capture the final message from a file. One `codex exec` call = one autonomous agent.

## Before launching: ask model + effort

If the user did NOT specify, ask them **which model** and **which reasoning effort** before running. Do not silently pick.

- **Effort** values: `minimal` | `low` | `medium` | `high`. High = slowest/smartest.
- **Model**: e.g. `gpt-5.6-sol`. If unsure of available names, the config defaults live in `~/.codex/config.toml` (`grep -i model ~/.codex/config.toml`); offer that default. Always pass `--model` explicitly — the model misreports its own name, so never rely on self-report and never assume the config default is what the user wants.

## The autonomous invocation (copy this)

```bash
codex exec \
  --model gpt-5.6-sol \
  -c model_reasoning_effort=high \
  --sandbox workspace-write \
  --cd /path/to/worktree \
  --skip-git-repo-check \
  -o /tmp/codex-result.txt \
  "Self-contained task prompt. State the goal, constraints, and what 'done' looks like." \
  < /dev/null
# then read the result:
cat /tmp/codex-result.txt
```

- `< /dev/null` — prevents Codex blocking on stdin ("Reading additional input from stdin").
- `-o FILE` (`--output-last-message`) — writes ONLY the agent's final message; read this instead of scraping the transcript.
- `--sandbox workspace-write` — lets Codex edit files under its workdir + `/tmp`. Use `read-only` for research/review-only tasks.
- `approval: never` is automatic under `codex exec`; it won't stop to ask. Long tasks may run minutes — set a generous `Bash` timeout (180000+).

## Quick reference

| Need | Flag |
| --- | --- |
| Pick model | `--model <name>` / `-m` |
| Reasoning effort | `-c model_reasoning_effort=high` |
| Working dir (worktree) | `--cd <dir>` / `-C` |
| Read-only (no edits) | `--sandbox read-only` |
| Capture final message | `-o <file>` |
| Structured output | `--output-schema <schema.json>` (final msg is JSON matching it) |
| Stream progress events | `--json` (JSONL to stdout) |
| Extra writable dir | `--add-dir <dir>` |
| Run outside git repo | `--skip-git-repo-check` |
| Code review of a diff | `codex exec review --base <branch>` (or `--uncommitted` / `--commit <sha>`) |

## Structured output (for verifier / decision agents)

When you need a machine-readable verdict (e.g. pass/fail), pass a JSON Schema; the final message is validated JSON:

```bash
echo '{"type":"object","properties":{"pass":{"type":"boolean"},"reason":{"type":"string"}},"required":["pass","reason"],"additionalProperties":false}' > /tmp/verdict.json
codex exec --model gpt-5.6-sol -c model_reasoning_effort=high --sandbox read-only \
  --output-schema /tmp/verdict.json -o /tmp/verdict.txt \
  "Does the current diff satisfy: <criteria>? Return pass + reason." < /dev/null
```

## Fanning out (flock over Codex)

To run tickets/tasks in parallel through Codex, give each its own **git worktree** and point `--cd` at it — no branch collisions. Each `codex exec` is one background `Bash` call. This is the same shape as the `flock-agents` skill, but the per-ticket implementation agent is a `codex exec` call instead of a Claude subagent. Prefer running each as a background Bash job; poll their result files. Keep concurrency modest (Codex hits its own rate limits).

## Common mistakes

- **Blocking on stdin** — always append `< /dev/null` (or pipe the real stdin you intend).
- **Scraping the transcript for the answer** — use `-o FILE`; it isolates the final message.
- **Trusting the model's self-reported name** — it said "GPT-5" when run as gpt-5.6-sol. Set `--model` explicitly and trust that.
- **Wrong sandbox** — `read-only` can't write files (research/review); `workspace-write` is needed to implement. Don't use `danger-full-access` / `--dangerously-bypass-*` unless the environment is externally sandboxed.
- **Too-short Bash timeout** — high-effort runs take minutes; raise the timeout and/or run in background.
- **Vague prompt** — Codex won't ask follow-ups under `exec`. The prompt must be fully self-contained: goal, constraints, definition of done.
- **Skipping the model/effort question** — if the user didn't specify, ask first.

## Verify it's usable

`codex login status` (should say logged in). If `which codex` fails, Codex isn't installed — stop and tell the user.
