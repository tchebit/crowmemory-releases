---
name: crow-memory-vendor-failover-pi
description: Set up and use CrowMemory's watch daemon (`crow-memory-mcp watch`) in pi so unattended work can fail over between model vendors through one shared handoff channel — for when a provider's quota or token runs out mid-task and you want a different vendor to pick up the remaining work without editing any config. Load this before wiring `pi -p` into a watch.toml agent, or before pushing a handoff meant for a specific vendor.
---

# CrowMemory — vendor failover via the watch daemon (pi)

> Route unattended work to whichever model vendor is actually available, decided per push, not per config edit.

## When to use

You're running `pi -p` unattended (via CrowMemory's watch daemon, a cron job,
or any headless dispatch) and want a way to redirect work to a backup vendor
the moment your primary one is rate-limited, out of quota, or otherwise down
— without stopping to edit a config file first.

## Background: the watch daemon

`crow-memory-mcp watch` is a small daemon that lives **outside** any agent's
process tree (an MCP server is a child of its agent — it dies when the agent
does, so nothing inside MCP can ever start a *new* agent). The daemon watches
handoff channels and spawns a configured CLI command when work has been
waiting long enough. That's the whole trick: delivery to an agent that isn't
currently running comes from the OS layer, not from MCP.

Config lives in a `watch.toml` beside your other CrowMemory configuration.
Two sections matter here: `[agents.*]` (how to start a thing) and
`[[route]]` (which channel wakes which agent).

## Why pi is a good fit for this

`pi` speaks multiple providers from one binary (`--provider`, `--model`), and
`pi -p` runs its tools autonomously when spawned headless — there's no
separate approval flag to get right for Bash calls versus file edits. That
matters here specifically: some other one-shot CLIs gate file edits and shell
execution under *different* flags, and if you only authorize one of them, a
headless run can silently do nothing on the other and exit 0 — no error, no
retry, just a wasted wake. `pi -p` doesn't have that split.

## Set up two agents behind one failover channel

```toml
[watch]
# Daemon-wide cap across ALL channels, not per-channel — size it for how
# many wakes you actually want overlapping, not just this one route.
max_concurrent = 3

[agents.pi-primary]
cmd        = ["pi", "-p", "--provider", "anthropic", "--model", "claude-sonnet-5", "--no-session", "{prompt}"]
unattended = true

[agents.pi-backup]
cmd        = ["pi", "-p", "--provider", "zai", "--model", "glm-5.3-flash", "--no-session", "{prompt}"]
unattended = true

[[route]]
channel = "failover"
agent   = "pi-primary"   # used only when a push omits `to`
```

`unattended = true` is a required, explicit opt-in per agent — the daemon
refuses to start without it, on purpose. A woken process has no terminal;
this is where you write down that you meant that.

## Pushing to it — the part that actually matters

```
handoff_push(
  channel: "failover",
  to: "pi-backup",                 # picks the vendor for THIS push
  note: "<task + working context>",
  actions: [
    "<step 1>",
    "<step 2>",
    "push the result to 'failover-<task-id>-result' as a topic, then stop"
  ]
)
```

- **Always set `to` explicitly** when the reason you're using this at all is
  "vendor X is unavailable." Omitting it falls back to the route's default
  agent — which may be exactly the vendor you're trying to avoid.
- **Do not override the route's prompt.** `entry.to` only reaches the right
  process because the daemon's *default* wake prompt renders `{as_flag}` into
  ` --as 'pi-backup'` for a directed entry. A hand-written prompt that
  hardcodes a plain `handoff pop <channel>` (no `--as`) wakes the right CLI
  but hands it a command that can never see its own entry.
- **Write real `actions`.** The default prompt is generic ("do what the
  entry asks"); it won't invent a task from a vague note.
- **The channel is a queue** — one push, one pop, gone. Have the woken agent
  report to a separate **topic** channel so you can read the result later
  from any session.

## Example

```
Situation: your Anthropic quota just got capped for the next two hours,
mid-refactor.

1. handoff_push(channel: "failover", to: "pi-backup",
     note: "Finish renaming `fetchUser` to `getUser` across src/. Working
            dir: <project root>.",
     actions: ["grep for remaining fetchUser call sites",
               "rename them, run the test suite",
               "push a summary to 'failover-rename-result' as a topic",
               "stop"])
2. The watch daemon wakes `pi-backup` (glm) within its debounce window —
   no session needs to be open on your end.
3. Later: handoff_read("failover-rename-result") to see what got done.
4. When your Anthropic quota resets, push future work with
   to: "pi-primary" (or just omit `to`, since it's the route's default).
```
