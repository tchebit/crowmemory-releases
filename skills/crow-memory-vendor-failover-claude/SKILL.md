---
name: crow-memory-vendor-failover-claude
description: Set up and use CrowMemory's watch daemon (`crow-memory-mcp watch`) with Claude Code so unattended work can fail over to a different agent/vendor through one shared handoff channel — for when Claude's usage or rate limit runs out mid-task and another CLI should pick up the remaining work without editing any config. Load this before wiring `claude -p` into a watch.toml agent, or before pushing a handoff meant for a specific vendor.
---

# CrowMemory — vendor failover via the watch daemon (Claude Code)

> Route unattended work to whichever agent is actually available, decided per push, not per config edit.

## When to use

You're dispatching `claude -p` unattended (via CrowMemory's watch daemon, a
cron job, or any headless flow) and want a way to redirect work to a backup
agent the moment Claude is rate-limited or out of usage — without stopping to
edit a config file first.

## Background: the watch daemon

`crow-memory-mcp watch` is a small daemon that lives **outside** any agent's
process tree — an MCP server is a child of its agent and dies with it, so
nothing inside MCP can ever start a *new* agent. The daemon watches handoff
channels and spawns a configured CLI command once work has been waiting long
enough. Delivery to an agent that isn't currently running comes from the OS
layer, not from MCP.

Config lives in a `watch.toml` beside your other CrowMemory configuration.
Two sections matter here: `[agents.*]` (how to start a thing) and
`[[route]]` (which channel wakes which agent).

## The permission-flag gotcha, specifically for `claude -p`

A headless `claude -p` has no terminal to answer an approval prompt. Every
agent you register **must** carry a flag that pre-authorizes what it's about
to do, and the daemon requires you to mark it `unattended = true` in writing
— it will not guess this for you.

The gotcha: `--permission-mode acceptEdits` only pre-authorizes **file
edits**. It does not cover Bash tool calls. A headless run that needs to
execute a shell command (for example, the CrowMemory `handoff` CLI itself)
will silently decline that call, then exit 0 having done nothing — no error,
no crash, just a wake that accomplished nothing. If your task needs to run
shell commands, use a flag that actually covers that, such as
`--dangerously-skip-permissions` (understand what that authorizes before
using it — it removes every approval gate, not just the Bash one).

## Set up primary + backup behind one failover channel

Claude Code's CLI talks to one vendor, so "failover" here means routing to a
**different agent entirely** for the backup — any CLI with a non-interactive,
one-shot mode works (`gemini -y -p`, `codex exec`, `pi -p --provider ...`,
even a plain script):

```toml
[watch]
# Daemon-wide cap across ALL channels, not per-channel — size it for how
# many wakes you actually want overlapping, not just this one route.
max_concurrent = 3

[agents.claude-primary]
cmd        = ["claude", "-p", "--dangerously-skip-permissions", "{prompt}"]
unattended = true

[agents.gemini-backup]
cmd        = ["gemini", "-y", "-p", "{prompt}"]
unattended = true

[[route]]
channel = "failover"
agent   = "claude-primary"   # used only when a push omits `to`
```

## Pushing to it — the part that actually matters

```
handoff_push(
  channel: "failover",
  to: "gemini-backup",             # picks the agent for THIS push
  note: "<task + working context>",
  actions: [
    "<step 1>",
    "<step 2>",
    "push the result to 'failover-<task-id>-result' as a topic, then stop"
  ]
)
```

- **Always set `to` explicitly** when the reason you're using this at all is
  "Claude is unavailable right now." Omitting it falls back to the route's
  default agent — which may be exactly the one you're trying to avoid.
- **Do not override the route's prompt.** `entry.to` only reaches the right
  process because the daemon's *default* wake prompt renders `{as_flag}` into
  ` --as 'gemini-backup'` for a directed entry. A hand-written prompt that
  hardcodes a plain `handoff pop <channel>` (no `--as`) wakes the right CLI
  but hands it a command that can never see its own entry.
- **Write real `actions`.** The default prompt is generic ("do what the
  entry asks"); it won't invent a task from a vague note.
- **The channel is a queue** — one push, one pop, gone. Have the woken agent
  report to a separate **topic** channel so you can read the result later
  from any session.

## Example

```
Situation: Claude Code hits its usage limit mid-task, two hours from reset.

1. handoff_push(channel: "failover", to: "gemini-backup",
     note: "Finish renaming `fetchUser` to `getUser` across src/. Working
            dir: <project root>.",
     actions: ["grep for remaining fetchUser call sites",
               "rename them, run the test suite",
               "push a summary to 'failover-rename-result' as a topic",
               "stop"])
2. The watch daemon wakes `gemini-backup` within its debounce window — no
   Claude session needs to be open on your end.
3. Later: handoff_read("failover-rename-result") to see what got done.
4. When Claude's usage resets, push future work with to: "claude-primary"
   (or just omit `to`, since it's the route's default).
```
