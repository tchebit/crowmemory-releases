# crow-memory-skill-free-default

> Default skill for CrowMemory Free edition — teaches AI agents to use all free tools effectively.

## When to use

This is the baseline skill for any AI agent using CrowMemory Free. Apply it at the start of every session to establish memory-aware behavior. It covers semantic search, knowledge graph building, memory lifecycle, and session continuity.

## Instructions

You have access to CrowMemory, a persistent memory system that survives across sessions, projects, and tools. Use it proactively — don't wait to be asked.

### Core behaviors

**Search before answering.** Before making assumptions about the project, search memory first. Call `recall` with specific technical keywords related to the topic. This prevents duplicating work and surfaces past decisions.

**Store with structure.** When saving a decision, bug fix, task, or question, pass `memory_type` (context, decision, task, bug, question, agent), `tags`, and optionally `priority` to `remember`. Omit them for a quick unstructured note.

**Link as you go.** After storing a memory that relates to an existing one, call `link_memories` to connect them. Use relationship types like `fixes`, `answers`, `depends-on`, `supersedes`, `implements`, `related-to`. This builds a knowledge graph over time. Use `unlink_memories` to undo a bad link.

**Keep memory clean.** When you store an updated version of something already in memory, archive the old version with `forget` (default is a soft delete/archive, not permanent). Stale entries pollute future recall results. `remember` also warns you when a near-duplicate memory already exists, so you can archive it instead of piling on.

**Walk trails when investigating.** When tracing a bug or decision, call `get_related_memories` on any memory you find. Follow the links to build the full picture.

### Pinned memories

**Pin critical context.** Use `pin_memory` for memories that should always be accessible — project conventions, architecture decisions, team agreements, active sprint goals. Pinned memories surface via `get_pinned` without needing a search query; they are not injected into `recall` results automatically.

**Start sessions with pinned context.** At the beginning of a session, call `get_pinned` to load always-relevant context before doing any work. This is the fastest way to recover orientation after a `/compact` or `/clear`, since Claude Code drops conversation state at those points but `get_pinned` reloads it explicitly. See [Auto-load pinned memories with a SessionStart hook](../README.md#claude-code-auto-load-pinned-memories-on-session-start) for wiring this up automatically.

**Keep pins fresh.** Call `pin_memory(memory_id, pinned=false)` to unpin outdated memories. A cluttered pin list defeats the purpose. Pins should be a curated shortlist, not a dump.

### Memory organization

**Scope by project.** CrowMemory shares one brain across all projects. `remember` scopes memories to the current project automatically (a `project:<slug>` tag and a `[slug]` text prefix), so you don't need to add these by hand. When recalling, include the project name in the query if you want to disambiguate.

**Use tags and types for filtering.** Pass `tags` and/or `memory_type` to `recall` to scope results — e.g. all memories tagged `auth`, or all memories of type `decision`. This works for status reports and audits ("show me every open task").

### Session continuity

**Drop breadcrumbs.** After each significant action (bug fixed, feature shipped, decision made), call `remember` with `memory_type="context"`. A `session:YYYY-MM-DD` tag is added automatically. At session start, recall recent breadcrumbs to pick up where you left off.

**Store pending work as tasks.** Use `memory_type="task"` for work that isn't finished yet. Recover it later with `recall(memory_type="task")`.

### Memory lifecycle

- `forget` — remove a memory: soft-delete/archive by default (recoverable via direct lookup, hidden from search), `permanent=true` for irreversible hard delete
- `update_memory` — modify the text and/or metadata (type, tags, priority) of an existing memory in place
- `get_memory` — retrieve full details of a specific memory by ID
- `list_memories` — browse all memories with pagination

### Temporal awareness

Every recall result includes a `created_at` timestamp. Use it to resolve time questions: "last session" means the most recent cluster by date, "yesterday" means the previous calendar day (UTC), "recently" means the last 7 days.

### Security

Never store passwords, API keys, tokens, or credentials in memory. Reference them by location (e.g. "see .env") — never by value.

### Keep it concise

Each memory has a 10,000 character limit. The embedding model indexes the first ~512 tokens — anything beyond is stored but invisible to search. For longer content, split into chunks and link them into a trail.

## Available tools

- `remember` — store a memory, optionally typed/tagged/prioritized; auto-scopes to project + session, warns on near-duplicates
- `recall` — search (semantic, filterable by `memory_type` and/or `tags`)
- `update_memory` — modify text and/or metadata of an existing memory
- `forget` — archive (default) or permanently delete a memory
- `list_memories` — paginated memory listing
- `get_memory` — retrieve a specific memory
- `link_memories` / `unlink_memories` — create or remove relationships between memories
- `get_related_memories` — walk the knowledge graph
- `pin_memory` / `get_pinned` — mark a memory as always-relevant and retrieve pinned memories

Run `crow-memory-mcp init` once per project to write the tier-specific system prompt file.

## Example

```
User: "How does our auth system work?"

Agent:
1. recall("auth system authentication JWT middleware")
2. Finds 3 related memories about auth decisions
3. get_related_memories on the most relevant one
4. Discovers linked memories about token refresh and session handling
5. Answers the question using memory context, not guessing
```
