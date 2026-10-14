# ADR-020: LLM-Based Context-Aware Chunking (Opt-In & Production Wiring Status)

| Field | Value |
|---|---|
| **Status** | `Accepted` (Implemented; not wired into connector batch sync path) |
| **Date** | 2026-10-14 |
| **Deciders** | AI Team |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

SprintStart ingests and chunks heterogeneous documents (Markdown, text, PDFs, code) to build a retrieval-augmented generation (RAG) corpus. While code files use AST-based Tree-sitter parsing ([ADR-014](adr-014-treesitter-ast-based-code-chunking.md)), prose documents (text, Markdown, PDFs) historically relied solely on paragraph-accumulation and character limits (`chunk_text`).

Standard paragraph-accumulation has two limitations:
1. **Topic fragmentation:** Boundary cuts occur when character budgets are reached rather than at natural semantic topic shifts.
2. **Loss of situating context:** Standalone chunks extracted from the middle of a large document lose their enclosing context (e.g. which section or module they describe), reducing retrieval precision and grounding quality.

An LLM can analyze the document to identify semantic topic boundaries and generate concise situating context headers (Anthropic-style Contextual Retrieval). However, calling an LLM during ingestion introduces latency, token costs, rate limits, and failure modes.

## Decision Drivers

- **Retrieval relevance:** Keep semantically related paragraphs together and provide situating context headers.
- **Fail-safe determinism:** Ingestion must never fail if the LLM backend is unreachable or produces invalid output.
- **Performance & throughput:** High-volume batch ingestion runs (e.g. syncing entire GitHub repositories or Confluence spaces) must remain fast and cost-effective.
- **Independent feature toggles:** Allow callers to independently enable semantic boundary detection, context block generation, or both.

## Considered Options

- **Option A – LLM context-aware chunking by default for all prose ingestion.** Every text and PDF document is passed through an LLM to determine boundaries and generate context headers.
- **Option B – Opt-in LLM context-aware chunking with independent flags and deterministic fallback.** Implemented behind `semantic_boundaries` and `contextualize` flags on `IngestRequest`, falling back to deterministic `chunk_text` on any error or when disabled.
- **Option C – Purely deterministic chunking without LLM participation.** Rely solely on character- and paragraph-length accumulation (`chunk_text`).

## Decision

**Chosen option: Option B** – Implement LLM context-aware chunking (`ingestion/context_aware_chunker.py`) as an opt-in strategy with two independent toggles (`semantic_boundaries` and `contextualize`), protected by strict character gates and automatic fallback to `chunk_text`.

## Current Implementation & Production Wiring Status

### Implemented Features

1. **Dual Independent Toggles (single LLM call):**
   - `semantic_boundaries`: The LLM selects paragraph groupings using integer indices (`boundaries: [int]`).
   - `contextualize`: The LLM generates a concise (1–2 sentence) situating header for chunks that genuinely benefit from it (`context_blocks: {"<chunk_index>": str}`).
2. **Context Block Bounds:**
   - Minimum threshold (`_CONTEXT_MIN_CHARS = 15`): Drops trivial noise blocks (e.g. `"Intro"`).
   - Maximum budget cap (`_CONTEXT_MAX_FRACTION = 0.25`): Truncates oversized context blocks to 25% of `chunk_size` so adequate budget remains for the chunk body.
3. **Safe Hard-Splitting:**
   - When `context_block + body` exceeds `chunk_size`, `_hard_split_with_context` re-prepends the context block to each sub-chunk and bounds `overlap_range` to the actual slice length.
4. **Deterministic Fallbacks:**
   - Falls back immediately to `chunk_text` if both flags are `False`, if text exceeds `CONTEXT_AWARE_CHUNKING_MAX_CHARS` (24,000 chars), if the LLM is unreachable, or if output parsing fails.

### Production Wiring Status

- **Single Upload Route (`POST /api/v1/ingest`):** **Wired.** The route inspects `body.semantic_boundaries` and `body.contextualize` on `IngestRequest`. If either flag is enabled, it passes `llm` to `parse()` and `parse_pdf()`.
- **Connector Batch Sync Route (`POST /api/v1/ingest/sync`):** **Not wired.** `ingest_run.py` calls `parse(filename, content_bytes)` without the `llm` parameter. All automated connector runs (GitHub, Confluence, Jira, Notion) process documents deterministically via `chunk_text`.

## Rationale

Batch connector runs often ingest hundreds of files per run. Running LLM-based chunking across entire repositories would drastically increase ingestion latency and token costs, and expose connector runs to LLM provider rate limits. Keeping the batch sync path deterministic protects core ingestion throughput, while keeping the LLM chunker available on `POST /ingest` allows targeted, high-value document ingestion with enhanced context.

## Pros and Cons of the Options

### Option A – Default LLM Chunking

- ✅ Maximizes retrieval quality across all ingested documents.
- ❌ High latency and cost; connector sync runs would be bottlenecked by LLM rate limits.
- ❌ Ingestion failure rate increases if LLM backend experiences transient outages.

### Option B – Opt-In with Deterministic Fallback (Chosen)

- ✅ Full feature availability for specialized/manual ingestion (`POST /ingest`).
- ✅ Zero latency or cost overhead on production connector batch syncs (`POST /ingest/sync`).
- ✅ Robust fallback hierarchy guarantees ingestion always succeeds.
- ❌ Automated connector runs currently do not benefit from context-aware headers.

### Option C – Purely Deterministic Chunking

- ✅ Minimal complexity and zero LLM cost.
- ❌ Loses the ability to evaluate and benchmark LLM-enhanced retrieval quality.

## Consequences

**Positive:**
- Context-aware chunking is fully implemented, unit-tested with `StubLLMClient`, and available for manual or API-driven document uploads.
- Production batch ingestion remains fast, cost-effective, and independent of LLM uptime.

**Trade-offs / Limitations:**
- Connector-ingested documents currently use standard paragraph-based chunking without situating context blocks.

**Follow-up actions:**
- Evaluate exposing `semantic_boundaries` / `contextualize` as configurable options on specific source connectors (e.g. selectively enabling contextualization for Confluence spaces or README files) once asynchronous background ingestion queues are available.

## Links

- [ADR Index](index.md)
- [ADR-014: AST-Based Code Chunking](adr-014-treesitter-ast-based-code-chunking.md)
- [ADR-016: Canonical Artifact Schema v1](adr-016-canonical-artifact-schema-v1.md)
