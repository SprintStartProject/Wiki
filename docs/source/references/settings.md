# Settings

Settings (`/settings`) is the personal configuration hub: user profile, appearance and access tokens on one page. It is bound to `/settings`, open to every permission group, and the old `/profile` route is a pure redirect to `/settings` (see [ADR-018](../explanations/adrs/adr-018-frontend-access-control-model.md)).

## Who sees what

- **Every permission group** — User Profile and Appearance.
- **PM, HR and ADMIN only** — Access Tokens; everyone else is not shown the section.
- The section navigation is built from the sections a user may see, so nobody is offered a section that is not on their page.

## Reaching a section

The section links are real anchors — `#profile`, `#appearance` and `#tokens` — so a section can be linked to directly. Clicking one scrolls the page to that section and records it in the URL; the sections are stacked on the page rather than hidden behind tabs, and the link row is horizontally scrollable on narrow screens. The sections appear in the same order as the links: profile, appearance, then tokens for everyone who sees it.

## Sections

### User Profile

Account details — first name, last name, email and avatar — with an explicit Save button that reflects unsaved changes. The account and password forms are the shared profile forms, so they behave exactly as they did on the former standalone profile page.

Password changes are not handled on the page itself: "Update Password" starts the authentication provider's `UPDATE_PASSWORD` flow and returns to Settings when it is done.

### Appearance

How SprintStart looks. Every control applies immediately and is remembered across visits — there is no save button in this section:

- **Theme** — Light, System or Dark.
- **Effects & animations** — the Aurora Background, with a glow-intensity slider, and the Card Tilt effect, each independently switchable.
- **Calm style notice** — when the system asks for reduced motion, SprintStart runs in its calm style and the animated background stays hidden; a notice in this section explains why and offers "Turn animations on".
- **Extras** — the Rocket Pet, which is off by default.

### Access Tokens

Credentials for the connected sources, used for ingestion — for example a GitHub Personal Access Token or an Atlassian credential. The list is source-filterable (default filter "In use", plus "All sources" and one entry per source). The default keeps sources with nothing stored out of the way; "All sources" reveals every connector the installation knows, and the add menu reaches them regardless of the filter.

Entries are named, and adding, rotating and deleting all happen inline in the row that owns them (deletion asks for confirmation).

**The section is shown to PM, HR and ADMIN only.** A plain `USER` never sees it: access tokens are the same credentials the ingestion pipeline runs on. The section renders the same unified list as the Access Management page in the admin area, so the two cannot drift apart.

## For developers

- `src/pages/SettingsPage.tsx` defines the three sections and the role gate (`PAT_ALLOWED_GROUPS`); the sections themselves live in `src/features/settings/components/`.
- The profile section reuses `useProfile` and the shared account/password forms; the token list comes from the access registry (`src/features/access/registry.tsx`) — a new connector adds its credential UI here by adding a registry entry.
- The route is part of the central access policy: `/settings` is open to every permission group, and `/profile` has its own redirect entry in `src/auth/accessPolicy.ts`.
