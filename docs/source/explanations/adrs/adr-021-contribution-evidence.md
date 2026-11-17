# ADR-021: Per-file contribution evidence

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-11-11 |
| **Deciders** | Backend Team |
| **Supersedes** | - |
| **Superseded by** | - |

---

## Context

Features that reason about who knows which part of a codebase, such as knowledge-loss risk, reviewer and successor recommendation, and expert lookup, need evidence per person and per file: who changed a file, when, how much, and who reviewed it.

Before this decision, ingestion did not keep that evidence:

- A commit artifact stored hash, message and date. The author name reached the commit event but was dropped when the event was mapped to an artifact, and the author's email was never read.
- A pull-request artifact stored the author login and counts. The changed file paths and the review comments' paths were fetched and then discarded, and individual reviewers were not kept.
- A file artifact stores the current content and path, with no history.

Commits are read from the local clone, so the missing commit data is available at no extra API cost for every provider. Review data is only available from the provider's pull-request API.

[ADR-016](adr-016-canonical-artifact-schema-v1.md) keeps source-specific details in the artifact's typed metadata, serialised into a text column, instead of adding tables per connector. The evidence described here is not source-specific, but it has to be queried by path and by person, which a text column cannot do efficiently.

## Decision Drivers

- Queries are by file path and by person over the whole history, for example "who knows these ten files" and "which files would be left without a knowledgeable person".
- The same shape must serve every provider that goes through the git ingestion engine.
- Commit-level data must not require provider API calls or rate limits.
- Author emails are personal data and must not become visible through existing APIs.
- A repository that is already ingested must be able to gain the data without being disconnected and reconnected.

## Considered Options

- **Option A** - Two relational tables owned by the `ingestion` module, `file_change` and `review_activity`, filled from the clone's log and from the provider's pull-request data
- **Option B** - Keep the per-file list inside each artifact's metadata, following ADR-016
- **Option C** - Store nothing new and compute evidence from the clones when it is needed

## Decision

**Chosen option: Option A** - Store contribution evidence in two dedicated tables owned by `ingestion`. Other modules read it through a small read API in `ingestion`, which is added together with the first feature that needs it.

## Rationale

Evidence is read far more often than it is written, and always by path or by person. Option A is the only option that lets the database answer those queries with an index. It keeps ADR-016's intent: connectors stay unaware of the tables, because the rows are produced from provider-neutral commit and review events, not by connector code.

Option B stays closest to ADR-016 but turns every query into a scan that parses JSON in a text column. Option C needs no schema change, but a clone is a cache that may be evicted or re-created, history walks over large repositories are slow when repeated per request, and review activity does not exist in a clone at all.

## Pros and Cons of the Options

### Option A - Dedicated tables owned by ingestion

- ✅ Path and person queries can use indexes.
- ✅ One schema for every provider; connectors only supply events.
- ✅ Rows can be re-derived from the history, so the tables are a rebuildable projection and a bug in them can be fixed by re-ingesting.
- ❌ Adds two entities to a module that so far has one canonical entity.
- ❌ Row count grows with commits times files touched; very large commits need a cap.
- ❌ Departs from ADR-016's "no new tables for source-specific data", although the data is not source-specific.

### Option B - Per-file list in artifact metadata

- ✅ No schema change and no new entity.
- ✅ Follows ADR-016 literally.
- ❌ Every path or person query scans and parses text.
- ❌ Metadata would grow with commit size and make artifact listings heavier.
- ❌ Mixes analysis data into the artifact that is also sent to AI indexing.

### Option C - Compute from the clones on demand

- ✅ No storage and no schema change.
- ❌ Depends on a cache that is allowed to disappear.
- ❌ Repeats an expensive history walk for each request.
- ❌ Cannot provide review activity.

## Consequences

**Positive:**
- Knowledge-loss scoring, successor recommendation and expert lookup read one indexed source.
- A reader never needs to know which provider a repository came from.
- Existing repositories gain the data by reading their history again from the beginning. Each connection remembers whether that has happened, so the re-read runs once and a failed attempt is retried.

**Negative / Trade-offs:**
- Author emails are stored. They are to be readable only through the `ingestion` module's read API, for project managers, and appear in no existing response.
- Commit evidence exists for GitHub and Bitbucket. Review evidence exists for GitHub pull requests; Bitbucket follows the same service once its pull-request data is mapped.
- Rows belong to their commit or pull-request artifact and are deleted with it through the foreign key. A deleted project needs no cleanup of its own: deleting a project only removes its links to artifacts, so the rows simply stop being reachable from it.
- The tables start with indexes on the owning artifact and the repository only. The first reader adds the index its queries need.

## Rules for the evidence

- **Merge commits:** the commit is ingested, but it lists no files, because the merged commits already carry the same changes.
- **Bulk commits:** a commit that touches more than a configured number of files is stored with no file rows. The threshold is configuration. The commit is not marked as bulk, because no consumer needs the difference between "no files" and "too many to list" yet; a marker can be added when one does.
- **Renames:** followed, so a moved file keeps its authors.
- **Reviews:** a reviewer is recorded for every file of a pull request they reviewed, with their latest submitted review. Drafts and the pull request author's own comments are not reviews. A pull request with more files than one page of the provider's API holds has no review rows, because a reviewer cannot be said to know files that were cut off.
- **Identity:** a commit email is matched to a person automatically, through GitHub's no-reply address (which contains the login) or through the email of a registered user. When that finds nobody, a project manager can confirm an alias by hand. The automatic match never overrides a confirmed alias. The matching lives with the feature that consumes the evidence, not in `ingestion`.

## References

- [ADR-016](adr-016-canonical-artifact-schema-v1.md) Canonical artifact schema v1
- [ADR-007](adr-007-sprintstart-backend-event-based-communication.md) Event-based communication between modules
