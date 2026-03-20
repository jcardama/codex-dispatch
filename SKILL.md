---
name: codex-dispatch
description: >
  Dispatch tasks to Codex via the Codex MCP server from Claude Code. Use when asking Codex to
  review Claude's code changes (git diff, unstaged changes, regressions), asking Codex to execute
  an implementation plan Claude wrote, or any task where Codex should work inside a specific repo.
  Triggers on phrases like "get Codex to review", "have Codex implement", "send this to Codex",
  "ask Codex to execute the plan", "Codex review", "Codex implement".
---

# codex-dispatch

Dispatch work to Codex via the Codex MCP server with correct parameters every time.

## Prerequisites

Before dispatching, verify the Codex MCP server is available. If it isn't connected, surface the error to the user rather than silently failing.

## Required Parameters (never omit)

| Parameter | Rule |
|-----------|------|
| `cwd` | Always the **target repo** directory — never the workspace or plan location |
| `approval-policy` | Always `"never"` — prevents Codex from blocking on approval prompts |
| `sandbox` | `"read-only"` for reviews, `"workspace-write"` for implementation tasks |
| `prompt` | Self-contained — include all context inline (file paths, diffs, plan text) |

## Workflows

### 1. Code Review (diff / unstaged changes)

```
codex - codex (MCP)(
  prompt: "Review the git diff of unstaged changes for <FOCUS>.\n\nRun: git diff <FILE1> <FILE2> ...\n\n<SPECIFIC QUESTIONS>",
  sandbox: "read-only",
  approval-policy: "never",
  cwd: "<ABSOLUTE_PATH_TO_REPO>"
)
```

**Prompt must include:**
- The exact `git diff` command to run (with explicit file paths)
- Specific questions or concerns to address
- Any relevant context (what was changed and why)

### 2. Execute an Implementation Plan

When Claude has written a plan outside the repo (e.g. in a workspace file):

```
codex - codex (MCP)(
  prompt: "<PASTE FULL PLAN TEXT HERE — do not reference external files>\n\nWork only within the cwd. Do not modify files outside it.",
  sandbox: "workspace-write",
  approval-policy: "never",
  cwd: "<ABSOLUTE_PATH_TO_REPO>"
)
```

**Key rule:** The plan text must be **inlined** into `prompt`. Never pass a path to a plan file — Codex won't read files outside `cwd`. Copy the full plan content into the prompt string.

### 3. General Task in a Repo

```
codex - codex (MCP)(
  prompt: "<TASK DESCRIPTION — fully self-contained>",
  sandbox: "workspace-write",
  approval-policy: "never",
  cwd: "<ABSOLUTE_PATH_TO_REPO>"
)
```

## After Dispatching

Once Codex responds:
1. **Read the output** — summarize findings or changes for the user
2. **Flag issues** — if Codex reports errors, blocked files, or missing context, surface them immediately
3. **Confirm completion** — only say "done" if Codex explicitly confirms it finished (not just that it started)

## Error Recovery

If Codex gets stuck, errors, or produces no output:
- Check that `cwd` exists and is the correct repo path
- Verify the prompt is fully self-contained (no references to files outside `cwd`)
- Retry with a more explicit prompt, or break the task into smaller steps
- If the MCP server appears unresponsive, tell the user and suggest restarting it

## Path Notes

- **Windows repos accessed from WSL:** use the WSL mount path (e.g. `/mnt/c/Users/<you>/Projects/MyRepo`) — not the Windows path
- **Remote machine repos** (e.g. macOS via SSH): Codex must run on the machine where the repo lives; dispatching to a remote path from a local MCP server won't work
- Always use **absolute paths** for `cwd`

## Checklist Before Dispatching

- [ ] Codex MCP server is connected and available
- [ ] `cwd` points at the **target repo** — absolute path, correct OS format
- [ ] `approval-policy: "never"` is set
- [ ] `sandbox` matches the task (`read-only` for review, `workspace-write` for implementation)
- [ ] `prompt` is fully self-contained (no references to external files or paths)
- [ ] For reviews: `git diff` command lists explicit file paths
- [ ] For plans: full plan text is inlined, not referenced by path
