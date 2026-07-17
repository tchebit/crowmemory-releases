# crow-memory-skill-pro-default

> Default skill for CrowMemory Pro edition — extends the free skill with hybrid search, pair sync, and event tracking.

## When to use

This is the baseline skill for any AI agent using CrowMemory Pro or Teams. It includes everything from the free skill (pinned memories — `pin_memory`, `unpin_memory`, `get_pinned` — are free-tier tools, see the free skill) plus Pro-exclusive capabilities: hybrid search for precise keyword matching, encrypted pair sync for team collaboration, and structured event tracking for observability.

## Instructions

You have access to CrowMemory Pro, a persistent memory system with advanced search, collaboration, and observability features. Use it proactively — don't wait to be asked.

### Core behaviors (inherited from Free)

**Search before answering.** Before making assumptions, search memory first. Call `recall` with specific technical keywords. This prevents duplicating work and surfaces past decisions.

**Store with structure.** Use `remember_with_metadata` for decisions, bugs, tasks, and questions. Use plain `remember` only for quick notes.

**Link as you go.** Call `link_memories` after storing related memories. Use relationship types: `fixes`, `answers`, `depends-on`, `supersedes`, `implements`, `related-to`.

**Keep memory clean.** Archive old versions with `archive_memory` when storing updates.

**Walk trails.** Call `get_related_memories` to follow decision chains and debug trails.

### Pro: Hybrid search

**Use hybrid search for exact matches.** When searching for specific identifiers, error codes, function names, or technical terms that must match exactly, use `hybrid_recall` instead of `recall`. It combines vector similarity with exact keyword matching for precise retrieval.

**When to use which:**
- `recall` — conceptual queries: "how does auth work", "database architecture decisions"
- `hybrid_recall` — exact terms: "ERR_CONNECTION_REFUSED", "CoolingStorage", "vector_id u64", specific UUIDs

**Fallback strategy.** If `hybrid_recall` returns nothing, fall back to `recall` with broader keywords. If `recall` returns too much noise, switch to `hybrid_recall` with the exact term.

### Pinned memories (free tier — see free skill for details)

Use `pin_memory` / `get_pinned` for always-on context and `unpin_memory` to keep the pin list curated. These are free-tier tools, not Pro-exclusive.

### Pro: Pair Sync

**Share context with teammates.** Use `pair_share` to gather relevant memories, encrypt them with a PIN, and upload to the relay. Share the token via chat — communicate the PIN verbally or via a separate secure channel.

**Receive shared context.** Use `pair_receive` with the token and PIN to download and merge the shared bundle into your local brain. Duplicates (same UUID) are skipped — local always wins.

**Use cases:**
- Onboard a new team member: share 20 memories about your auth API, schema, and deployment
- Hand off a task: share the decision trail and current state
- Sync across machines: share your project context from laptop to desktop

### Memory organization

**Scope by project.** Include a `project:<id>` tag when storing and prefix text with `[project-id]`. Include the project name in queries. For `hybrid_recall`, the keyword component matches the prefix naturally.

**Use tags for filtering.** `recall_by_tag` scopes searches to specific tags. AND mode (default) requires all tags. OR mode explores across categories.

**Use types for classification.** `recall_by_type` finds all memories of a given type — decisions, bugs, tasks, questions, agent profiles.

### Session continuity

**Drop breadcrumbs.** After significant actions, store a breadcrumb with `remember_with_metadata` using type `context`, tag `session:YYYY-MM-DD`, and the project tag. At session start, recall recent breadcrumbs to resume.

**Start with pins + breadcrumbs.** Call `get_pinned` for always-on context, then recall recent breadcrumbs for session-specific state.

### Memory lifecycle

- `archive_memory` / `restore_memory` — soft-delete and recover
- `forget` — permanent delete
- `update_memory` — modify text in place
- `pin_memory` / `unpin_memory` — mark as always-relevant (free tier)
- `get_pinned` — retrieve all pinned memories (free tier)

### Temporal awareness

Every recall result includes `created_at` (RFC 3339). Use it for time queries: "last session" = most recent cluster, "yesterday" = previous UTC day, "recently" = last 7 days.

### Security

Never store credentials in memory. Reference by location only.

### Keep it concise

10,000 char limit per memory. Embedding indexes ~512 tokens. Split long content into chunks and link them.

## Available tools

**Free tools (17):**
- `remember`, `remember_with_metadata` — store memories
- `recall`, `recall_by_type`, `recall_by_tag` — search memories
- `list_memories`, `get_memory` — browse and retrieve
- `update_memory`, `forget` — modify and delete
- `archive_memory`, `restore_memory` — lifecycle management
- `link_memories`, `get_related_memories` — knowledge graph
- `pin_memory`, `unpin_memory`, `get_pinned` — pinned memory management
- `init` — project initialization

**Pro tools (3):**
- `hybrid_recall` — vector + keyword fusion search
- `pair_share`, `pair_receive` — encrypted context sharing

## Example

```
User: "What was that ERR_SSL_PROTOCOL_ERROR we fixed last week?"

Agent:
1. hybrid_recall("ERR_SSL_PROTOCOL_ERROR") — exact keyword match
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
