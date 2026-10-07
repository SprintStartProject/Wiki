# How to implement a new connector

## Purpose

This guide describes how to add a new data source (a connector such as GitHub, Bitbucket, Jira, Confluence or Notion) to the SprintStart frontend, and what the backend and the AI service have to provide for it.

The registry and the per-connector UI code live in `sprintstart-frontend/src/features/data-ingestion/connectors/`. The data ingestion page, the add-source flow, the create-project wizard, the knowledge base and the chat read the connectors from this registry. Most of the code does not branch on the source system. A new connector is therefore one folder plus one registry entry, and the compiler lists most of what is still missing. A few places still name connectors one by one and are not caught by the compiler; step 6 lists them.

```{note}
The registry comes with the data ingestion refactor of the frontend. Paths and type names below refer to `sprintstart-frontend`.
```

---

## Prerequisites

- The backend connector exists and is reachable through the endpoints listed under "What backend and AI have to provide" below.
- You know which of the existing connectors is closest to the new one. Copy it, do not start from nothing:
  - **Jira**: an instance with a stored credential, ingested as a whole, with a schedule endpoint.
  - **Confluence**: a connection owned by one project, with a synchronous sync and a schedule stored on the connection record.
  - **GitHub**: a repository shared between projects, discovered through a picker, with per-resource sync times.
  - **Bitbucket**: the GitHub shape for a Bitbucket Cloud workspace. Repositories are shared between projects and discovered through the same picker, and the stored Atlassian credential is shared with Jira and Confluence. It syncs pull requests only.
  - **Upload**: no upstream at all, so no actions.

---

## What a connector consists of

```text
connectors/
├── sourceSystems.ts          # SOURCE_SYSTEMS and the SourceSystem type, the only definition
├── types.ts                  # ConnectorDefinition and the supports it is made of
├── registry.ts               # CONNECTORS, CONNECTOR_LIST, getConnector and derived lists
├── draft.ts                  # DraftSource union, what the add-source flow stages
├── actionContext.ts          # requireProjectId, shared by the actions
├── projectSources.ts         # shared connection support for connectors that are project sources (GitHub, Bitbucket, Upload)
├── knowledgeBaseLink.ts      # builds the knowledge base link of a source from knowledgeBase.scopeOf
└── <connector>/
    ├── definition.ts         # the ConnectorDefinition
    ├── draft.ts              # staged-source type, duplicate check, connect call
    ├── DraftForm.tsx         # add-source form, holds its own state, reports drafts
    ├── DetailsSection.tsx    # optional, the connector's rows in the source details panel
    └── *MetadataView.tsx     # optional, view for artifacts that are metadata only (knowledgeBase.metadataView)
```

A definition declares the following. The right column says who reads it, so you can tell what breaks when a field is wrong.

| Field | Declares | Read by |
| --- | --- | --- |
| `meta` | label, card name, noun, icon, description, backend connector id, optional `meta.connector` (`label`, `description`) for the Manage connectors modal | cards, type grid, run labels, Manage connectors modal |
| `identity`, `toDetails`, `fallbackBackendStatus` | the card's key and name, its source specific details, its status when there is no status row | `createDataSource`, the one generic card mapper |
| `runFallback` (optional) | how a card without a status row reads its artifact total and sync time off the newest run | `createDataSource` |
| `resourceSyncTimes`, `runReferences` | the last sync time per resource, the references a run carries as its `sourceId` | source details panel, run labels |
| `connections` | how the connector's own records of a project's sources are loaded | `useIngestionData` |
| `actions` | `update` or `manualSync`, `unlink`, `setEnabled`, `schedule`. An action that is left out is not offered | source details panel, project wide sync settings |
| `DetailsSection` | the connector's identity rows in the details panel, or `null` | `SourceDetailsPanel` |
| `runFilter` | which run query parameter (`repositoryId` or `sourceRef`) scopes the history to one source, and the value | run history filter |
| `draft` | add-source form, staged row wording, duplicate check, connect call, optional owner assignment | add-source flow, staged list, wizard review |
| `knowledgeBase` | facet label and order, icon, "Open in ..." text, optional `repositoryFacet` flag, optional metadata view, optional scope for the link from a source to its artifacts | knowledge base filters and URL state, artifact viewer, source details panel |
| `chat` | whether the chat source filter offers it, how a cited URL is matched to it | chat composer, citation drawer |

---

## Steps

### 1. Add the source system

Add the name to `SOURCE_SYSTEMS` in `sourceSystems.ts`. The compiler now reports what has to follow: the `CONNECTORS` record in `registry.ts` (add the connector's connection record type to `ConnectionOf`) and `DraftSourceOf` in `connectors/draft.ts`.

### 2. Write the service

Put the backend calls in `src/services/sources/<connector>Service.ts`. Document every exported function (purpose, parameters, failure behaviour) and let failures surface. Definitions call the service, components never do.

### 3. Create the folder

Create `connectors/<connector>/` with `definition.ts`, `draft.ts` and `DraftForm.tsx`. Add `DetailsSection.tsx` if the connector has identity rows of its own.

- `draft.ts` holds the staged source type (`DraftSourceBase & { type: "<SYSTEM>"; ... }`), a creator that starts from `newDraftBase()`, a function that decides whether two staged rows are the same source, and the connect call. The connect call returns `DraftConnectOutcome`, so a connector that can link to an existing connection says so (`wasReused`).
- `DraftForm.tsx` keeps its own state and reports the drafts it currently describes through `onDraftsChange`: none while it is incomplete, one for a plain form, several for a multi select. Leaving the screen throws the state away, so there is nothing to reset. Use `CredentialSlot` for an inline "add credential" companion.

### 4. Write the definition

A trimmed example, modelled on Jira:

```ts
export const exampleConnector: ConnectorDefinition<ExampleConnectionDto, ExampleDraftSource> = {
  meta: {
    system: "EXAMPLE",
    connectorId: "example", // lowercase, equal to the id the backend lists
    label: "Example",
    name: "Example Workspace",
    noun: { singular: "workspace", plural: "workspaces" },
    icon: Box, // any IconComponent, a hand made SVG works too (see components/icons/BitbucketIcon.tsx)
    description: "Indexes pages from Example workspaces.",
  },
  chat: {
    filterable: false, // switch on once the AI service accepts the system
    matchesCitationUrl: (url) => url.includes("example.com"),
  },
  knowledgeBase: { label: "Example", facetOrder: 5, icon: Box, linkLabel: "Open in Example" },
  DetailsSection: ExampleDetailsSection,
  runFilter: { param: "sourceRef", valueOf: (source) => source.sourceId },

  draft: {
    DraftForm: ExampleDraftForm,
    formHint: "Pick a credential and a workspace, then add it to the list.",
    title: (draft) => draft.displayName,
    detail: (draft) => draft.credentialName,
    isSame: isSameExampleDraft,
    connect: connectExampleDraft,
  },

  connections: {
    scope: "example",
    live: true, // the records change while a run is in flight
    load: (projectId) => getExampleConnections(projectId),
  },

  actions: {
    update: {
      isAvailable: (source) => exampleConnectionOf(source) !== null,
      unavailableReason: "Updates need the workspace id.",
      async run(source) {
        // call the service, throw with a readable message on failure
      },
    },
    unlink: {
      isAvailable: (source) => exampleConnectionOf(source) !== null,
      removalHint: "The pages it already ingested are kept.", // what removal costs
      async run(source, context) {
        // use requireProjectId(context, "removing it") for project owned connections
      },
    },
    // setEnabled, schedule and manualSync only if the backend offers them
  },

  identity: (status, connection) => ({
    sourceId: connection?.id ?? status?.sourceId ?? "",
    name: status?.displayName ?? connection?.name ?? "",
  }),
  fallbackBackendStatus: (connection) => (connection?.sourceEnabled === false ? "DISABLED" : "CONNECTED"),
  toDetails: (_status, connection) => ({ system: "EXAMPLE", workspace: connection ?? null }),
  resourceSyncTimes: () => [],
  runReferences: (details) => (details.system === "EXAMPLE" && details.workspace ? [details.workspace.id] : []),
};
```

Rules of thumb:

- Anything the connector has no use for is left out (`actions.schedule`, `runFilter`, `DetailsSection: null`), not stubbed with a function that does nothing.
- `ConnectionSupport.failureMessage` decides whether a failed load is reported. Leave it out where the endpoint may be off limits to some users (an HR user without the PM role), and the cards are built without the records instead.
- Set `knowledgeBase.repositoryFacet: true` if the connector's artifacts belong to a repository. The knowledge base then offers the repository facet and keeps the `repos` URL param while the connector is among the selected sources (`hasRepositoryFacet` in `registry.ts`). A `knowledgeBase.scopeOf` that returns repositories only works together with this flag, because the link it builds is otherwise stripped of its `repositories` narrowing on arrival.
- The functions in the definition types are declared as methods on purpose. That keeps a definition for one connection record assignable to the registry's view of all of them.

### 5. Register it

Add the definition to `CONNECTORS` in `registry.ts`. The type of the record makes a missing entry a compile error. Derived lists (`CHAT_SOURCE_SYSTEMS`, `SCHEDULED_SOURCE_SYSTEMS`, `KNOWLEDGE_BASE_SOURCE_ORDER`, the chat citation matching) follow from the flags in the definition.

### 6. Finish the parts that still name connectors one by one

Some places still know the connectors individually, and the compiler does not report them. Keep this list short when touching the code.

**Data ingestion**

1. **`SourceDetails` union** in `data-ingestion/types.ts`, plus an accessor in `sourceDetails.ts` (`exampleConnectionOf`). The definition's actions read the connector's details through it.
2. **Card assembly**: `buildSources.ts` builds the cards from the status rows and the connection records, and `useIngestionData` needs one more connection query next to the existing ones.
3. **Run details**: `buildOriginRow` in `RunDetailsPanel.tsx` shows the connector specific origin of a run (owner, domain, space). Add a row if the connector has one.

**Knowledge base and chat**

4. **URL state**: `knowledge-base/hooks/useKnowledgeBaseUrlState.ts` parses the `repositories` param only while GITHUB is among the selected sources, and `format` only while UPLOAD is. A connector whose `knowledgeBase.scopeOf` returns repositories gets a link from `knowledgeBaseLink.ts` whose `repositories` narrowing is dropped on arrival, unless the parser accepts it for the new system.
5. **Facets**: `knowledge-base/hooks/useKnowledgeBase.ts` offers the repository facet options only for GITHUB. Extend it if the connector's artifacts belong to a repository.
6. **Artifact metadata**: `knowledge-base/githubMetadata.ts` parses the metadata of GitHub and Bitbucket artifacts. It also reads the repository an artifact belongs to (shown in the artifact list and viewer) and decides in `matchesRepository` which artifacts survive a repository selection. That function names GITHUB and BITBUCKET one by one and lets every other system through. A connector with `repositoryFacet: true` has to be added there, and a connector with its own metadata needs a parser and a view (`knowledgeBase.metadataView`) of its own.
7. **Chat sources**: `chatbot/hooks/useAvailableSources.ts` always offers UPLOAD (`ALWAYS_AVAILABLE`) and the enabled connectors on top. Only touch it if the new system has no connector to enable.
8. **GITHUB fallbacks**: `ingestionService` and `knowledgeService` fall back to GITHUB where the backend sends no `sourceSystem`. The backend has to send it for the new connector, otherwise its rows show up as GitHub.

The knowledge base URL state and the repository facet need no code of their own: they follow the `knowledgeBase.repositoryFacet` flag of the definition (step 4).

### 7. Tests

`registry.test.ts` already checks that every source system has a complete definition, so a missing label or icon fails there. Add:

- service tests and MSW handlers for the new service,
- the connector's cases in `SourceDetailsPanel.test.tsx` (actions, details section) and `AddSourceModal.test.tsx` (form, staging, connect),
- an a11y test for the form,
- a case in `citationArtifact.test.ts` if the connector claims citation URLs.

### 8. Run the checks

From `sprintstart-frontend`:

```bash
npm run format:check
npm run lint
npm run build
npm run unit
npm run a11y
```

On Node 25 or newer, run the test commands with `NODE_OPTIONS=--no-experimental-webstorage`.

---

## What backend and AI have to provide

The frontend only works against the following contract. Agree on it before the connector UI is built.

**Backend**

- The connector appears in `GET /api/v1/connectors` with a lowercase id equal to `meta.connectorId`. Its sources can be switched with `PATCH /api/v1/connectors/{id}/sources/status`.
- `GET /api/v1/ingestion-sources/status` returns one row per source of the project, with `sourceSystem`, `sourceId`, health, counters and `artifactCount`.
- Runs carry `sourceSystem` and can be filtered by `repositoryId` or `sourceRef`, whichever the definition's `runFilter` names.
- Endpoints to connect, to update or sync, and to unlink a single source. If the connector has a schedule, either its own endpoint or the schedule on the connection record (as Confluence does).
- Artifacts carry the new `sourceSystem` and a `sourceUrl`, so the knowledge base facet and the "Open in ..." link work.
- The `sourceSystem` enum accepts the new value everywhere the frontend receives or sends it (status rows, runs, artifacts, filters).

**AI service**

- For the chat source filter (`chat.filterable: true`) the AI service has to accept the system in its source filter. Leave the flag off until that change is deployed.
- Cited sources have to carry a URL that the connector's `chat.matchesCitationUrl` can recognise. A citation names only the URL and the title of the artifact, so this is how the chat knows which connector an opened citation belongs to.

---

## Loading and refreshing

The data ingestion page loads through `useIngestionData` (TanStack Query). The status rows, the project's latest runs, the run table's page and each connector's connection records are separate queries under `queryKeys.ingestion`. They are reloaded every few seconds while a run is in flight and for a minute after a connect or an update. After a mutation, call the page's `refreshIngestionData`, which invalidates `queryKeys.ingestion.all()`, instead of reloading by hand.

---

## Checklist

- [ ] Source system added, registry entry present, `ConnectionOf` and `DraftSourceOf` extended
- [ ] Service with documented functions and tests
- [ ] `definition.ts`, `draft.ts`, `DraftForm.tsx` (and `DetailsSection.tsx` if needed)
- [ ] `SourceDetails` union, accessor, card assembly, connection query, run origin row
- [ ] Knowledge base wiring checked (`repositoryFacet` flag, artifact metadata) and chat source list
- [ ] Tests extended, a11y test for the form, all five checks green
- [ ] Backend contract confirmed, AI filter flag set only after the AI change is deployed
