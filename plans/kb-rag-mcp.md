# KB RAG — MCP Server with Local Embeddings

Semantic search over the team knowledge base at `~/dev/infra/knowladge-base`, exposed as an MCP tool so Claude Code can retrieve relevant context automatically before reading source code.

## Motivation

The KB holds ADRs, architecture notes, and cross-project decisions that are not derivable from source code. Currently, `/readkb` reads whole files on demand. As the KB grows, a vector search layer lets Claude retrieve only the relevant chunks without reading everything.

## Design

```
knowladge-base/          (markdown source)
        │
        ▼
  watcher + chunker      (split by markdown section, tag with project/type)
        │
        ▼
  sqlite-vec             (local vector store, single file, no daemon)
        │
        ▼
  MCP server             (exposes tools to Claude Code)
```

**Embedding model:** `nomic-embed-text` via Ollama (local, no API cost).

**Chunking strategy:** split each file at `## ` headings, keep frontmatter as metadata. Each chunk stores: `project`, `file` (`decisions` | `architecture` | `glossary` | `README`), `heading`, `text`.

**MCP tools exposed:**
- `search_kb(query, k?)` — semantic search, returns top-k chunks with source pointers
- `get_kb_entry(project, file)` — direct fetch of a full file

**Indexing:** file watcher re-indexes changed files on save. Full re-index on startup if schema or model changes.

## Stack

- Language: Python or Rust (Rust preferred — fits homelab toolchain)
- Vector store: `sqlite-vec` (single `.db` file, no separate process)
- Embeddings: Ollama HTTP API (`nomic-embed-text`)
- MCP: `mcp` crate (Rust) or `mcp` Python SDK
- File watching: `notify` (Rust) or `watchdog` (Python)

## Deployment

Run as a systemd user service on the homeserver. Claude Code on any machine connects via MCP over WireGuard (or locally if on the homeserver).

Alternatively, run as a local process per-machine and sync the `.db` file — sqlite-vec is a single file, easy to rsync.

## Claude Code Integration

Register in `~/.claude/settings.json` under `mcpServers`. Once connected, update the `/readkb` skill to call `search_kb` instead of reading files directly, and add a `CLAUDE.md` rule:

> Before exploring repo source files for architectural context, call `search_kb` with the relevant project/topic first.

## Open Questions

- Rust vs Python — Rust matches homelab toolchain but Python has better MCP + embedding ecosystem maturity right now
- Local vs homeserver — local process per machine is simpler; homeserver is a single source of truth
- Sync strategy if multi-machine: rsync the `.db`, or expose MCP over WireGuard
