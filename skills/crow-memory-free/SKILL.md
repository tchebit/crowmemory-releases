---
name: crow-memory-free
description: Use CrowMemory's memory tools correctly on the Free edition — search before answering, store with the right memory_type, link related memories into a decision trail, keep memory clean, and hand work to another agent over a handoff channel. Load this at the start of any session using mcp__crow-memory__* tools.
---

# CrowMemory — Free edition

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

### Handoff Channels — passing work between agents

`recall`/`remember` are long-term memory. **Handoff Channels** are the opposite: ephemeral messages that hand a task from one agent to another (or to a human) without copying memory ids between chat sessions. They live in an isolated store and **never appear in `recall`**.

A handoff is an envelope: `refs` (memory ids it points to, resolved live) + `note` (a short instruction). Store durable content with `remember`, then reference its id in `refs` — don't copy content into the handoff. The channel name is the whole address — channel names are global across every project sharing a brain.

**Pick the mode by how it's consumed:**
- One agent should take it, then it's done → push with `mode="queue"` (default); the other agent claims it with `handoff_pop` (first matching taker wins).
- Many agents each see it, or you need replay / request-response (PING/PONG) → push with `mode="topic"`; consumers `handoff_read` without consuming, tracking position with a per-channel watermark (`since` → reuse the returned `next_since`).
- **Don't watermark a queue** — `pop` claims it; watermarks are for topics.

**A pop is a loan, not a transfer.** The entry is *leased* to you (default 300s), not deleted. The claim ends when you pop that channel again (which acks what you were holding), when you call `handoff_ack`, when you call `handoff_nack` to give it back, or when the lease lapses — in which case the entry is redelivered to someone else. So a crash loses nothing. If a popped entry shows `deliveries` greater than 1, an earlier consumer abandoned it: check for half-finished work before starting over. If you know you can't do the work, `handoff_nack` rather than walking away.

Entries expire after a TTL (default 72h — push first, start the consumer later). On a queue, `to`/`as` address an entry to a role so agents sharing a folder don't self-pop. A `key` gives latest-value-wins (same-key push supersedes); a keyed push with no payload retracts (tombstone). Put concrete follow-ups in `actions` and unknowns in `unresolved` rather than burying them in `note`. A handoff belongs to the operator who pushed it — pass `shared: true` to let anyone on the brain claim it. An empty pop/read explains why, distinguishing "nothing here" from "everything is leased to someone else", and lists the other live channels.

**Sessions can hand over by themselves.** If `crow-memory-mcp install-hooks` has been run in a project, ending a session writes a handoff on `session:<project>` and the next session receives it before its first prompt. When such a block is already in your context, act on it — don't pop that channel looking for another copy, it has already been consumed.

> `crow-memory-mcp init` also installs a dedicated **crow-memory-handoff** skill with the full model. That skill is versioned with the server, so prefer it over this summary if the two ever disagree.

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
- `handoff_push` — queue or broadcast an ephemeral handoff to another agent; envelope of `refs` + `note`, plus `actions` / `unresolved`; `mode="queue"` (one taker) or `mode="topic"` (broadcast); `shared: true` to let any operator claim it
- `handoff_pop` — claim the oldest entry off a queue channel (first matching taker wins). Leased, not deleted — see the loan rule above; `lease_secs` to hold it longer
- `handoff_ack` / `handoff_nack` — finish a claim (delete it) or hand it straight back. Optional: popping the same channel again acks what you held
- `handoff_read` — read a topic (or peek a queue) without consuming; watermark with `since`/`next_since`
- `handoff_channels` — list channels with their mode, `depth` (claimable now) and `in_flight` (claimed, not yet acked)

Run `crow-memory-mcp init` once per project. It writes the tier-specific system prompt file and installs the **crow-memory-handoff** skill. Run `crow-memory-mcp install-hooks` as well to have sessions hand over to each other automatically.

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
