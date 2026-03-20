# codex-dispatch

An agent skill that dispatches tasks to Codex via the Codex MCP server — with enforced parameters for `cwd`, `sandbox`, and self-contained prompts. Covers code review, plan execution, and general repo tasks.

## What it does

When you ask your agent to "get Codex to review the unstaged changes" or "have Codex execute this plan", `codex-dispatch` kicks in and makes sure every call to Codex is correct:

- **`cwd`** always points at the target repo, not wherever the agent is running from
- **`approval-policy: "never"`** so Codex never blocks waiting for a human
- **`sandbox`** is set appropriately (`read-only` for reviews, `workspace-write` for implementation)
- **Prompts are self-contained** — plan text is inlined, never referenced by path

## Installation

### Claude Code (Official Marketplace)

If this skill is listed on the official Claude plugin marketplace:

```
/plugin install codex-dispatch@claude-plugins-official
```

### Claude Code (via this repo)

Register the marketplace first:

```
/plugin marketplace add jcardama/codex-dispatch
```

Then install:

```
/plugin install codex-dispatch@codex-dispatch
```

### Manual

Copy `SKILL.md` into your agent's skills directory:

```bash
cp SKILL.md ~/.claude/skills/codex-dispatch/SKILL.md
# or
cp SKILL.md ~/.agents/skills/codex-dispatch/SKILL.md
```

## Usage

Trigger phrases (automatically recognized):

- `"get Codex to review the unstaged changes"`
- `"have Codex implement this plan"`
- `"send this to Codex"`
- `"ask Codex to execute the plan"`
- `"Codex review"` / `"Codex implement"`

## Workflows

| Workflow | Sandbox | When to use |
|----------|---------|-------------|
| Code review | `read-only` | Review git diff, unstaged changes, regressions |
| Execute plan | `workspace-write` | Codex implements a plan Claude wrote |
| General task | `workspace-write` | Any scoped task within a repo |

See [`SKILL.md`](SKILL.md) for full parameter reference and checklists.

## Requirements

- [Codex](https://github.com/openai/codex) with MCP server enabled
- Claude Code (or any agent that supports the skills protocol)

## Sponsor

If this skill has saved you time and you're so inclined, [buying me a coffee](https://buymeacoffee.com/jcardama) or [sponsoring on GitHub](https://github.com/sponsors/jcardama) is always appreciated. Thanks!

## License

MIT
