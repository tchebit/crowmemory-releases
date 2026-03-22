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
| Hybrid search (FTS5) | — | Yes | Yes |
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

1. Download the binary for your platform from [Releases](https://github.com/tchebit/crowmemory-releases/releases)
2. Make it executable: `chmod +x crow-memory-mcp-free-*`
3. Move to your PATH: `sudo mv crow-memory-mcp-free-* /usr/local/bin/crow-memory-mcp`
4. Add to your MCP client — for Claude Code:
   ```bash
   claude mcp add -s user crow-memory -- crow-memory-mcp
   ```

## Community Skills

CrowMemory supports community-contributed skills — reusable prompt patterns that teach AI agents specialized workflows on top of CrowMemory.

Browse available skills in the [`skills/`](./skills) folder, or **contribute your own** by opening a PR.

See [`skills/CONTRIBUTING.md`](./skills/CONTRIBUTING.md) for the format and guidelines.

## Links

- [Releases](https://github.com/tchebit/crowmemory-releases/releases)
- [Website](https://crowmemory.ai)
- [Report Issues](https://github.com/tchebit/crowmemory-releases/issues)
