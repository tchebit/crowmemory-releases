# crow-memory-skill-pro-default

> Default skill for CrowMemory Pro edition — extends the free skill with hybrid search, pair sync, and event tracking.

## When to use

This is the baseline skill for any AI agent using CrowMemory Pro or Teams. It includes everything from the free skill (pinned memories — `pin_memory`, `get_pinned` — are free-tier tools, see the free skill) plus Pro-exclusive capabilities: hybrid search fusion, encrypted pair sync for team collaboration, and structured event tracking for observability.

## Instructions

You have access to CrowMemory Pro, a persistent memory system with advanced search, collaboration, and observability features. Use it proactively — don't wait to be asked.

### Core behaviors (inherited from Free)

**Search before answering.** Before making assumptions, search memory first. Call `recall` with specific technical keywords. This prevents duplicating work and surfaces past decisions.

**Store with structure.** Pass `memory_type`, `tags`, and `priority` to `remember` for decisions, bugs, tasks, and questions. Omit them for quick notes.

**Link as you go.** Call `link_memories` after storing related memories. Use relationship types: `fixes`, `answers`, `depends-on`, `supersedes`, `implements`, `related-to`. Use `unlink_memories` to undo a link.

**Keep memory clean.** Archive old versions with `forget` (soft delete by default) when storing updates. `remember` also warns on near-duplicate content.

**Walk trails.** Call `get_related_memories` to follow decision chains and debug trails.

**Hand off work.** Use Handoff Channels (`handoff_push` / `handoff_pop` / `handoff_read` / `handoff_channels`) to pass an ephemeral task to another agent or a human — a queue for one-taker relay, a topic for broadcast or request/response. These are free-tier tools; see the free skill for the full model (envelope of `refs`+`note`, queue-vs-topic, watermarks, TTL, `to`/`as`, dedup `key`).

### Pro: Hybrid search is automatic

There is a single search tool, `recall`. With FTS5 enabled and a Pro/Teams license, `recall` automatically fuses vector similarity with exact keyword matching (RRF) — you never choose between "semantic" and "hybrid," and there is no separate tool or parameter for it. Just use specific technical keywords (error codes, function names, identifiers) in your query and the fusion picks them up naturally.

### Pinned memories (free tier — see free skill for details)

Use `pin_memory` / `get_pinned` for always-on context. Call `pin_memory(memory_id, pinned=false)` to unpin. These are free-tier tools, not Pro-exclusive.

### Pro: Pair Sync

**Share context with teammates.** Use `pair_share` to gather relevant memories, encrypt them with a PIN, and upload to the relay. Share the token via chat — communicate the PIN verbally or via a separate secure channel.

**Receive shared context.** Use `pair_receive` with the token and PIN to download and merge the shared bundle into your local brain. Duplicates (same UUID) are skipped — local always wins.

**Use cases:**
- Onboard a new team member: share 20 memories about your auth API, schema, and deployment
- Hand off a task: share the decision trail and current state
- Sync across machines: share your project context from laptop to desktop

### Memory organization

**Scope by project.** `remember` auto-scopes memories to the current project (`project:<slug>` tag, `[slug]` text prefix) — no manual tagging needed. Include the project name in queries if you want to disambiguate; the keyword-fusion component of `recall` matches the prefix naturally.

**Use tags and types for filtering.** Pass `tags` and/or `memory_type` to `recall` — e.g. all memories tagged a feature area, or all memories of type `decision`/`bug`/`task`.

### Session continuity

**Drop breadcrumbs.** After significant actions, call `remember` with `memory_type="context"` — a `session:YYYY-MM-DD` tag is added automatically. At session start, recall recent breadcrumbs to resume.

**Start with pins + breadcrumbs.** Call `get_pinned` for always-on context, then recall recent breadcrumbs for session-specific state.

### Memory lifecycle

- `forget` — archive (default, recoverable via direct lookup) or `permanent=true` for hard delete
- `update_memory` — modify text and/or metadata (type, tags, priority) in place
- `pin_memory` / `get_pinned` — mark as always-relevant / retrieve pinned (free tier)

### Temporal awareness

Every recall result includes `created_at` (RFC 3339). Use it for time queries: "last session" = most recent cluster, "yesterday" = previous UTC day, "recently" = last 7 days.

### Security

Never store credentials in memory. Reference by location only.

### Keep it concise

10,000 char limit per memory. Embedding indexes ~512 tokens. Split long content into chunks and link them.

## Available tools

**Free tools:**
- `remember`, `recall` — store and search memories
- `list_memories`, `get_memory` — browse and retrieve
- `update_memory`, `forget` — modify and delete (archive or permanent)
- `link_memories`, `unlink_memories`, `get_related_memories` — knowledge graph
- `pin_memory`, `get_pinned` — pinned memory management
- `handoff_push`, `handoff_pop`, `handoff_read`, `handoff_channels` — agent-to-agent handoff channels

**Pro tools:**
- `pair_share`, `pair_receive` — encrypted context sharing

`recall`'s hybrid keyword+vector fusion is a Pro/Teams runtime behavior, not a separate tool.

## Example

```
User: "What was that ERR_SSL_PROTOCOL_ERROR we fixed last week?"

Agent:
1. recall("ERR_SSL_PROTOCOL_ERROR") — hybrid fusion matches the exact code
2. Finds the bug memory with root cause and fix
3. get_related_memories — discovers linked decision about cert rotation
4. Answers with full context: the error, root cause, fix, and related decision
```

```
User: "Share the auth context with the new developer"

Agent:
1. recall("auth authentication middleware JWT project:my-api")
2. get_related_memories on key results to gather the full trail
3. pair_share with the relevant memories
4. Returns token + PIN for the teammate to use with pair_receive
```
