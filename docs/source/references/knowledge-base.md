# Knowledge Base

The Knowledge Base (`/knowledge-base`) is the unified view of everything a project has indexed: artifacts imported from its sources (GitHub, Jira, Confluence) and files uploaded by hand through the Upload source. It is open to every permission group and always scoped to the globally selected project — a user without a project switcher falls back to their first assigned project.

## Browsing

- The list arrives one page at a time (20 by default; 50 or 100 selectable) as cards showing the artifact type, its source, the repository for GitHub artifacts, ingestion and last-change dates, and the AI-ingestion status.
- The artifact types are pull requests, issues, files, docs, commits and organization metadata.

## Filters and facets

All of these narrow the same list, and their counts sit next to the options:

- **Search** (`?q=`) over the artifacts.
- **Type tabs** — All, Pull requests, Issues, Files, Docs, Commits, Organization.
- **Source** — any combination of the connectors the project has.
- **Repository** (GitHub) — the repositories behind the project's GitHub artifacts; offered only while GitHub is among the selected sources.
- **Format** (Uploads) — PDFs, Markdown, Images, Other; offered only while Uploads is among the selected sources. A format narrows uploads only: connector artifacts always match, so "GitHub + Uploads + PDFs" cannot hide the GitHub rows.
- **Language** — the languages the project's artifacts are written in.
- **Sort** — Newest added, Recently changed or Title A–Z.
- **Updated** — a date window on last activity (the last change, otherwise the import).
- **Result range** ("1–20 of N artifacts"), a refresh control, and "Clear filters".

Sections without options (for example Repository on a project without GitHub artifacts) are hidden rather than shown empty.

## The URL is the state

The page keeps its whole view state in the query string, which makes every view shareable:

| Parameter | Holds |
|---|---|
| `tab` | The type tab |
| `q` | Search text |
| `sources` | Selected sources (comma-separated) |
| `repos` | Selected repositories (GitHub only) |
| `format` | Upload format (Uploads only) |
| `languages` | Selected languages |
| `page`, `size` | Pagination |
| `sort` | List order |
| `from`, `to` | The Updated range (`yyyy-MM-dd` each end) |
| `artifact` | The artifact open in the viewer drawer |

Parsing is "drop what cannot be honoured, never fail": unknown values are ignored and page and size are clamped, so a hand-edited or stale link still renders something sensible. Switching projects drops the parameters that describe that project's corpus (including the open artifact); `size` and `sort` are reading preferences and survive.

The `artifact` parameter follows the drawer in both directions — opening a document puts it in the URL, closing it takes it away — so a link to a document reopens it, and a link copied after closing does not.

## The artifact viewer drawer

Selecting a card opens the drawer beside the list:

- **Raw content** — Markdown rendered or as source (a Formatted/Source toggle), PDFs embedded with a download fallback, images inline. Artifacts without stored bytes (for example GitHub organization metadata) render the connector's own metadata view instead. When the backend sent a `sourceUrl`, the header offers the way to the original ("Open in GitHub", …).
- **Summarise** — an AI summary written by the AI service and streamed over SSE, so it appears as it is generated. A "Sources" list of the cited chunks sits beneath it.
- **Ask AI** — quotes the selected passage (or asks about the whole artifact) into a new chat conversation; it fills the composer and the reader sends it.
- **Delete** — offered for uploads, to PM and ADMIN only; the same control is available as a bulk action from the list while Uploads is the selected source.

## Uploads

Files enter the corpus through the **Upload source**, not through the Knowledge Base itself: the Upload connector's form (in the add-source flow or the project wizard) stages the selected files and uploads them when the source is connected. It accepts PDF, Markdown, TXT, PNG, JPG and WEBP up to 10 MB per file; ingestion runs in the background, and the files then appear here under the Uploads source, filterable by format.

## For developers

- `sprintstart-frontend/src/features/knowledge-base/` holds the components, the hooks (including `useKnowledgeBaseUrlState.ts`) and the facet definitions in `tabs.ts`; the page is `src/pages/KnowledgeBasePage.tsx`.
- The list and facet counts come from `knowledgeService.getArtifactPage` / `getArtifactFacets`, scoped to the selected project; content, summaries and deletions go through `knowledgeService` as well.
- To make a new connector's artifacts appear and filter here, see [How to implement a new connector](../how-to-guides/how-to-implement-a-new-connector.md).
