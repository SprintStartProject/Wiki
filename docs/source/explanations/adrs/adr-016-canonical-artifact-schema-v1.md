# ADR-016: Canonical artifact schema v1

| Field | Value                 |
|---|-----------------------|
| **Status** | `Accepted`            |
| **Date** | 2026-07-13            |
| **Deciders** | Backend Team, AI Team |
| **Supersedes** | -                     |
| **Superseded by** | -                     |

---

## Context

SprintStart ingests onboarding-relevant artifacts from multiple source systems, currently GitHub and uploaded files, with Jira represented as a planned source system. These sources produce different native payloads, but the onboarding and AI indexing flows need one stable representation for searching, authorization, deindexing, and downstream ingestion.

The ingestion module already maps source-specific events into `ArtifactCommand` objects and persists them as `Artifact` rows. The AI sync path then maps those persisted artifacts into `ArtifactAiIngestRequest`. The decision is to formalize this shape as canonical artifact schema v1 instead of letting each source define its own persistence and indexing contract.

## Decision Drivers

- Keep connector modules loosely coupled from ingestion, onboarding, and AI indexing.
- Provide one queryable artifact model for cross-source search and project-scoped access.
- Preserve stable source identity for idempotent ingestion and deletion handling.
- Support source-specific metadata without adding a new table or API contract for every connector.
- Keep the AI ingestion payload stable as sources are added.
- Allow incomplete or source-dependent fields without rejecting useful artifacts.

## Considered Options

- **Option A** - Canonical artifact schema with source metadata extension
- **Option B** - Source-specific artifact tables with per-source AI adapters
- **Option C** - Raw source payload store with canonical fields derived on demand

## Decision

**Chosen option: Option A** - Store all ingestible artifacts in one canonical v1 model with common identity, content, project, and indexing fields plus source-specific metadata.

## Rationale

The canonical schema gives onboarding, search, and AI indexing one contract while allowing connectors to keep source-specific translation at module boundaries. It matches the current ingestion design: GitHub and upload events are mapped into canonical commands, persisted as artifacts, and sent to AI through a source-neutral request shape.

The schema should treat `sourceSystem` plus a stable `sourceId` as the source identity, use content hashes where a source supports change detection, and keep optional fields nullable when a source cannot provide them consistently. Source-specific details belong in typed metadata serialized into the artifact record, not in downstream API-specific DTOs or connector-owned database tables.

## Pros and Cons of the Options

### Option A - Canonical artifact schema with source metadata extension

- ✅ Gives search, onboarding, and AI indexing one stable artifact contract.
- ✅ Keeps source-specific mapping inside connector or ingestion adapter code.
- ✅ Makes project-scoped artifact listing, content retrieval, and deindexing consistent.
- ✅ Allows new source systems to plug in by producing canonical commands.
- ❌ Requires disciplined schema versioning and migrations when canonical fields change.
- ❌ Pushes source-specific fields into metadata, which is less relationally constrained.
- ❌ Requires consumers to handle nullable fields for sources that cannot provide full content or URLs.

### Option B - Source-specific artifact tables with per-source AI adapters

- ✅ Preserves each source system's native structure and constraints.
- ✅ Reduces nullable columns because each table can model only its own fields.
- ❌ Forces search, onboarding, and AI indexing to understand every source-specific shape.
- ❌ Makes adding a new connector more expensive because it needs persistence, query, and AI adapter work.
- ❌ Increases cross-module coupling and makes cross-source pagination/filtering harder.

### Option C - Raw source payload store with canonical fields derived on demand

- ✅ Maximizes ingestion flexibility and preserves original source payloads.
- ✅ Reduces upfront schema work for new sources.
- ❌ Makes querying, filtering, authorization, and AI indexing depend on repeated transformations.
- ❌ Weakens validation and makes idempotent update detection harder.
- ❌ Delays schema problems until runtime consumers need consistent fields.

## Consequences

**Positive:**
- Connectors can publish or map source events into a small canonical command surface.
- AI indexing receives one payload shape across GitHub, uploads, and future sources.
- Artifact identity, content hashing, project association, and deindexing can be handled consistently.
- The ingestion module remains the owner of artifact persistence and does not expose connector internals.

**Negative / Trade-offs:**
- Canonical v1 changes require coordinated migrations, mapper updates, and AI contract updates.
- Metadata JSON must remain intentional and documented so it does not become an unvalidated catch-all.
- Some common fields must stay nullable because not all sources have URLs, body content, mime type, or update timestamps.
- Database migrations need to stay aligned with the entity shape as canonical fields evolve.

**Follow-up actions:**
- Document the canonical v1 field list, required fields, nullable fields, and enum extension process.
- Align Flyway migrations with the current v1 entity shape, including metadata and project association tables if they are intended to be part of v1.
- Add or update contract tests around source mappers and `ArtifactAiIngestRequest` whenever the canonical schema changes.

## Links

- [ADR Index](../../explanations/adrs/index.md)
