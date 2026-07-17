# crow-memory-skill-free-default

> Default skill for CrowMemory Free edition — teaches AI agents to use all 17 free tools effectively.

## When to use

This is the baseline skill for any AI agent using CrowMemory Free. Apply it at the start of every session to establish memory-aware behavior. It covers semantic search, knowledge graph building, memory lifecycle, and session continuity.

## Instructions

You have access to CrowMemory, a persistent memory system that survives across sessions, projects, and tools. Use it proactively — don't wait to be asked.

### Core behaviors

**Search before answering.** Before making assumptions about the project, search memory first. Call `recall` with specific technical keywords related to the topic. This prevents duplicating work and surfaces past decisions.

**Store with structure.** When saving a decision, bug fix, task, or question, use `remember_with_metadata` with a type (context, decision, task, bug, question, agent) and relevant tags. Use plain `remember` only for quick unstructured notes.

**Link as you go.** After storing a memory that relates to an existing one, call `link_memories` to connect them. Use relationship types like `fixes`, `answers`, `depends-on`, `supersedes`, `implements`, `related-to`. This builds a knowledge graph over time.

**Keep memory clean.** When you store an updated version of something already in memory, archive the old version with `archive_memory`. Stale entries pollute future recall results.

**Walk trails when investigating.** When tracing a bug or decision, call `get_related_memories` on any memory you find. Follow the links to build the full picture.

### Pinned memories

**Pin critical context.** Use `pin_memory` for memories that should always be accessible — project conventions, architecture decisions, team agreements, active sprint goals. Pinned memories surface via `get_pinned` without needing a search query; they are not injected into `recall` results automatically.

**Start sessions with pinned context.** At the beginning of a session, call `get_pinned` to load always-relevant context before doing any work. This is the fastest way to recover orientation after a `/compact` or `/clear`, since Claude Code drops conversation state at those points but `get_pinned` reloads it explicitly. See [Auto-load pinned memories with a SessionStart hook](../README.md#claude-code-auto-load-pinned-memories-on-session-start) for wiring this up automatically.

**Keep pins fresh.** Unpin outdated memories with `unpin_memory`. A cluttered pin list defeats the purpose. Pins should be a curated shortlist, not a dump.

### Memory organization

**Scope by project.** CrowMemory shares one brain across all projects. Always include a `project:<id>` tag when storing (e.g. `project:my-api`), and prefix the memory text with `[project-id]` so search captures it. When recalling, include the project name in the query.

**Use tags for filtering.** Call `recall_by_tag` to scope searches to specific tags (project, session, feature area). Default mode is AND (all tags must match). Use OR mode when exploring across categories.

**Use types for classification.** Call `recall_by_type` to find all memories of a specific type — all decisions, all bugs, all tasks. This is powerful for status reports and audits.

### Session continuity

**Drop breadcrumbs.** After each significant action (bug fixed, feature shipped, decision made), store a breadcrumb using `remember_with_metadata` with type `context`, a `session:YYYY-MM-DD` tag, and the project tag. At session start, recall recent breadcrumbs to pick up where you left off.

**Store pending work as tasks.** Use type `task` for work that isn't finished yet. Recover it later with `recall_by_type` or `recall_by_tag`.

### Memory lifecycle

- `archive_memory` — soft-delete outdated memories (recoverable)
- `restore_memory` — bring back archived memories when needed
- `forget` — permanently delete a memory (irreversible)
- `update_memory` — modify the text of an existing memory in place
- `get_memory` — retrieve full details of a specific memory by ID
- `list_memories` — browse all memories with pagination

### Temporal awareness

Every recall result includes a `created_at` timestamp. Use it to resolve time questions: "last session" means the most recent cluster by date, "yesterday" means the previous calendar day (UTC), "recently" means the last 7 days.

### Security

Never store passwords, API keys, tokens, or credentials in memory. Reference them by location (e.g. "see .env") — never by value.

### Keep it concise

Each memory has a 10,000 character limit. The embedding model indexes the first ~512 tokens — anything beyond is stored but invisible to search. For longer content, split into chunks and link them into a trail.

## Available tools

- `remember` — store unstructured text
- `remember_with_metadata` — store with type, tags, and priority
- `recall` — semantic similarity search
- `recall_by_type` — search filtered by memory type
- `recall_by_tag` — search filtered by tags
- `list_memories` — paginated memory listing
- `get_memory` — retrieve a specific memory
- `update_memory` — modify existing memory text
- `forget` — permanently delete
- `archive_memory` — soft-delete
- `restore_memory` — recover archived memory
- `link_memories` — create relationships between memories
- `get_related_memories` — walk the knowledge graph
- `pin_memory` / `unpin_memory` — mark a memory as always-relevant
- `get_pinned` — retrieve all pinned memories
- `init` — initialize CrowMemory in a project

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
