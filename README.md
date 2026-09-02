# pstack (Claude Code port)

Upstream: <https://github.com/cursor/plugins/tree/main/pstack> (Cursor plugin, MIT, by Lauren Tan).
This tree is that plugin retargeted to Claude Code. Skill bodies are upstream prose except
where the platform differs.

## Port map

Re-apply these when syncing a newer upstream version.

| Cursor | Claude Code |
| --- | --- |
| `Task` tool, `subagent_type: generalPurpose` | `Agent` tool, `subagent_type: general-purpose` |
| `run_in_background: true`, `readonly: false` | dropped; subagents already run in the background, `subagent_type: Explore` is the read-only variant |
| `environment: "cloud"` | `isolation: "worktree"` for writers, `isolation: "remote"` where enabled |
| `AskQuestion` | `AskUserQuestion` |
| `grok-4.6-fast-xhigh` | `sonnet` |
| `gpt-5.6-sol-max` | `fable` |
| `claude-fable-5-1-thinking-max`, `claude-opus-5-thinking-xhigh` | `opus` |
| `~/.cursor/rules/pstack-models.mdc` (`alwaysApply`) | `~/.claude/pstack-models.md` (read on demand by the skills) |
| `~/.cursor/projects/<slug>/agent-transcripts/<id>/<id>.jsonl` | `~/.claude/projects/<slug>/<session-id>.jsonl`, subagents in `<session-id>/subagents/**/agent-*.jsonl`; slug = workspace path with every `/` and `.` turned into `-` |
| `.cursor/skills/`, `~/.cursor/skills/` | `.claude/skills/`, `~/.claude/skills/` |
| Cursor built-in `create-skill` | the `skill-creator` skill |
| `deslop` from `cursor-team-kit` | `/simplify` over code, `unslop` over prose |
| `control-ui` / `control-cli` from `cursor-team-kit` | the repo's own `verify-*` skill, else `/webapp-testing`, `/agent-browser`, `/smoke`, `/run` |
| Cursor dashboard for agent status | `ListAgents` / `claude agents` |

Names normalized to kebab-case so Claude Code loads them: `Make Bot UI` → `make-bot-ui`,
`Poteto Mode` → `poteto-mode`, `Comment Sicko` → `comment-sicko`.

`make-bot-ui` was dropped: it drives Cursor-only surfaces (`update_state` routines,
`SendToUser` secret cards, the Cursor computer preview). Nothing here maps to it.

## Install

```
claude plugin marketplace add ~/.claude/pstack
claude plugin install pstack@pstack
```

Skills then answer as `/pstack:<name>` (`/pstack:why`, `/pstack:poteto-mode`, ...).

## Model configuration

`/pstack:setup-pstack` writes `~/.claude/pstack-models.md`, one line per role. Every skill
reads it and falls back to its inline default when a line is absent. Valid values are the
`Agent` tool's model enum (`opus`, `sonnet`, `haiku`, `fable`) plus `inherit-parent` / `auto`,
which mean "omit `model`".
