# CrowMemory MCP

Persistent memory for AI agents. Works with any MCP-compatible client — Claude Code, Zed, Cursor, Windsurf, Claude Desktop, and more.

**[Download the latest release](https://github.com/tchebit/crowmemory-releases/releases)**

---

## What it does

CrowMemory gives AI agents long-term memory that persists across sessions, projects, and tools. Agents remember what they learned, what they decided, and why — without you managing anything.

## Capabilities

| Capability | Description |
|---|---|
| **Semantic Memory** | Store and recall context using vector similarity search |
| **Hybrid Search** | Combine semantic and keyword search for precise retrieval (Pro) |
| **Knowledge Graph** | Link related memories into navigable decision trails |
| **Agent Profiles** | Save and activate reusable agent personas across sessions |
| **Pinned Memories** | Mark memories as always-on context, loaded explicitly via `get_pinned` |
| **Memory Lifecycle** | Archive, restore, pin, and manage memory over time |
| **Temporal Queries** | Ask for memories by time — "yesterday", "last session", "before March" |
| **Pair Sync** | Share encrypted context with teammates via secure relay (Pro) |
| **Audit & Traceability** | Full observability into every AI memory operation (Pro) |
| **Tag & Type Filtering** | Organize memories by type (decision, bug, task, question) and custom tags |
| **Credential Guardrail** | Secrets are never stored — referenced by location only |

## Editions

| | Free | Pro | Teams |
|---|---|---|---|
| Semantic memory + knowledge graph | Yes | Yes | Yes |
| Shared memory across agents | Yes | Yes | Yes |
| Pinned memories (`pin_memory`, `get_pinned`) | Yes | Yes | Yes |
| Hybrid search (semantic + keyword) | — | Yes | Yes |
| Pair Sync (encrypted sharing) | — | Yes | Yes |
| Audit & traceability | — | Yes | Yes |
| All MCP tools | — | Yes | Yes |
| Team brain sync | — | — | Yes |

**Free** — download below. **Pro & Teams** — [crowmemory.ai](https://crowmemory.ai)

## Audit & Traceability (Pro / Teams)

Every memory operation — store, recall, delete, update, archive, link, sync — produces a structured JSONL event with full context: who, what, when, and which project. Stream these events to **your own infrastructure** — any HTTP endpoint, SIEM, logging pipeline, or local binary collector.

**You control the data.** CrowMemory does not phone home. Events go where you configure them — your servers, your collectors, your rules.

```toml
# Example: stream events to your logging infrastructure
[tracking]
level = "info"
exclude_events = ["memory.recalled"]  # optional filtering

[tracking.remote]
url = "https://your-logging-server.com/ingest"
authorization = "Bearer your-token"
headers = { "X-Team" = "engineering", "X-Env" = "production" }
batch_size = 10
flush_interval_ms = 5000

[tracking.local]
command = "/usr/local/bin/your-collector"
args = ["--format", "json"]
```

**Use cases:**
- **AI usage metrics** — measure how agents use memory across teams and projects
- **Compliance auditing** — prove what AI agents stored, recalled, and deleted
- **Cost attribution** — track memory operations per team, project, or session
- **Incident forensics** — trace exactly what an agent knew and when

## Quick Start

### macOS / Linux (Homebrew)

```bash
brew tap tchebit/crowmemory
brew install crow-memory-mcp
```

Then add it to your MCP client — see [Install on Claude](#install-on-claude) below, or the docs for Zed, Cursor, Windsurf, etc.

### Manual download (all platforms, incl. Windows)

1. Download the binary for your platform from [Releases](https://github.com/tchebit/crowmemory-releases/releases)
2. Make it executable: `chmod +x crow-memory-mcp-free-*`
3. Move to your PATH: `sudo mv crow-memory-mcp-free-* /usr/local/bin/crow-memory-mcp`
4. Add it to your MCP client — see [Install on Claude](#install-on-claude) below, or the docs for Zed, Cursor, Windsurf, etc.

## Install on Claude

CrowMemory works with both Claude Code (CLI/IDE) and Claude Desktop. Pick the one you use — you can add it to both, they share the same local memory brain.

### Claude Code

Run once, from anywhere:

```bash
claude mcp add -s user crow-memory -- crow-memory-mcp
```

- `-s user` registers it for every project (drop it, or use `-s project`, to scope it to the current project instead).
- Verify it's connected: `claude mcp list` should show `crow-memory` as `connected`.
- Restart any running `claude` sessions to pick it up.

To use the Pro/Teams binary instead, point the same command at that binary's path (e.g. `-- /usr/local/bin/crow-memory-mcp-pro --enable-fts5`).

### Claude Desktop

Claude Desktop reads its MCP server list from a config file:

| OS | Path |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

Create the file if it doesn't exist, and add (or merge into) the `mcpServers` block:

```json
{
  "mcpServers": {
    "crow-memory": {
      "command": "/usr/local/bin/crow-memory-mcp"
    }
  }
}
```

If the binary isn't on your `PATH`, use its full path in `command`. For the Pro/Teams binary, add the flag it needs:

```json
{
  "mcpServers": {
    "crow-memory": {
      "command": "/usr/local/bin/crow-memory-mcp-pro",
      "args": ["--enable-fts5"]
    }
  }
}
```

Restart Claude Desktop completely (quit, not just close the window) for the new server to load. You should see `crow-memory` listed under the MCP/plugin icon in a new chat.

## Claude Code: Auto-load Pinned Memories on Session Start

`get_pinned` is not injected into `recall` results — it only surfaces pinned context when called explicitly. Claude Code also drops conversation state on `/compact`, `/resume`, and `/clear`, so wiring `get_pinned` into a `SessionStart` hook keeps your agent oriented at every one of those points, not just at first launch. This works on **every edition** — `pin_memory`, `unpin_memory`, and `get_pinned` are all free-tier tools.

### 1. Create the hook script

Save this as `~/.claude/hooks/crow-memory-recall.sh` (user-level, applies to all projects) or `.claude/hooks/crow-memory-recall.sh` (project-level):

```bash
#!/usr/bin/env bash
# SessionStart hook — injected into context on startup / resume / compact / clear.
cat <<'MSG'
[crow-memory protocol] Before doing anything else:
1. Call mcp__crow-memory__get_pinned — load pinned context (architecture, conventions, must-know facts).
2. Before implementing or answering about prior work, call mcp__crow-memory__recall
   (or mcp__crow-memory__hybrid_recall on Pro/Teams) with project-specific keywords.
3. After completing a unit of work, store it with mcp__crow-memory__remember_with_metadata
   and link_memories to related entries.
Do not ask whether to checkpoint — follow this protocol.
MSG
```

Make it executable:

```bash
chmod +x ~/.claude/hooks/crow-memory-recall.sh
```

### 2. Register the hook

Add to `~/.claude/settings.json` (user-level) or `.claude/settings.json` (project-level, checked into the repo so it applies to every contributor):

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact|clear",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/crow-memory-recall.sh"
          }
        ]
      }
    ]
  }
}
```

Use a repo-relative path (e.g. `bash .claude/hooks/crow-memory-recall.sh`) instead if the hook script is checked into the project rather than kept at the user level.

### 3. Pin what matters

```
pin_memory(memory_id: "...")   # mark architecture decisions, conventions, must-know context
get_pinned()                    # verify what's loaded
```

Keep the pin list to a curated shortlist — critical guardrails and conventions, not a dump of everything you've ever stored.

## Community Skills

CrowMemory supports community-contributed skills — reusable prompt patterns that teach AI agents specialized workflows on top of CrowMemory.

Browse available skills in the [`skills/`](./skills) folder, or **contribute your own** by opening a PR.

See [`skills/CONTRIBUTING.md`](./skills/CONTRIBUTING.md) for the format and guidelines.

## Links

- [Releases](https://github.com/tchebit/crowmemory-releases/releases)
- [Homebrew tap](https://github.com/tchebit/homebrew-crowmemory)
- [Website](https://crowmemory.ai)
- [Report Issues](https://github.com/tchebit/crowmemory-releases/issues)
