# ADR-018: Frontend Access Control Model

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-07-21 |
| **Deciders** | Frontend Team |
| **Supersedes** | - |
| **Superseded by** | - |

---

## Context

SprintStart uses Keycloak for authentication and authorization ([ADR-015](adr-015-use-keycloak-authentication-authorization.md)). There are four realm roles: `USER`, `PM`, `HR` and `ADMIN`. The backend validates every request and is the only place where access is actually enforced.

The frontend still has to decide what a user sees: which routes they can open, which sidebar entries, dashboard widgets and keyboard shortcuts are shown, and where a user lands after login. Showing a page that then fails with 403 on every request is a bad experience, and hiding a sidebar entry alone does not stop someone from typing the URL.

Two things make this more than a simple role check:

- **Project scope.** Most PM surfaces (PM dashboard, data ingestion, blueprints, team management, insights) show data for the globally selected project. A user with the PM role can manage some projects and only be a member of others. Holding the PM role is not enough to manage a project. The backend enforces this with `@projectAuth.canAccessProject`.
- **Several roles per user.** A user can hold more than one realm role, but the UI needs one answer to "what kind of user is this".

The first version of the frontend checked roles in individual pages and components ("Added role based visibility", June 2026). As more roles and pages were added, these checks drifted apart. In July 2026 the rules were moved into one access policy, and the project-manager scoping was added on top of it later.

Related customer decision (2026-09-29): a regular user belongs to exactly one project at a time, while PMs can be in several. This is why only PM, HR and ADMIN get the global project switcher.

## Decision Drivers

- One place that answers "may this user open this route", used by routing, sidebar, dashboard and shortcuts alike.
- Adding a protected route without registering its permissions should fail at compile time.
- PM access to project-scoped pages depends on managing the selected project, not on the role alone.
- The frontend never widens access beyond what the backend allows. It only hides what would fail anyway.
- Role information comes from the same source the backend uses, so both sides agree.

## Considered Options

- **Option A** - Role checks inside each page and component
- **Option B** - Read roles from the Keycloak token directly in the frontend
- **Option C** - Central route-based access policy with project-manager scoping, backend stays authoritative
- **Option D** - Fine-grained permissions (capabilities) delivered by the backend per user

## Decision

**Chosen option: Option C** - A central, typed access policy in `src/auth/accessPolicy.ts` decides route visibility from the user's permission group and, for PMs, from whether they manage the selected project. The backend remains the only enforcement point.

## Rationale

A central policy keeps every surface consistent. The sidebar, the router guards, the dashboard catalog and the keyboard shortcuts all call the same `canAccessRoute`, so a route can no longer be visible in one place and blocked in another.

Making the policy route-based and typed means the rules live in a `Record<AppRoute, PermissionGroup[]>`. A new entry in the `AppRoute` union without a permission entry does not compile.

The permission group is taken from the backend profile (`GET /api/v1/users/me`), not from the Keycloak token. The backend reduces a user's roles to one effective group with the priority `ADMIN > HR > PM > USER`. Reading it from the backend keeps both sides on the same answer. It also avoids duplicating role mapping logic in the frontend.

Project-manager scoping is an extra rule on top of the role check rather than a separate permission system. It applies only to the PM role and only to routes that are scoped to the selected project. It can only narrow access, never widen it.

Fine-grained capabilities (Option D) would be more precise, but they need a new backend contract for every action. The current needs are covered by groups plus the manager rule. Individual actions inside a page (for example ADMIN-only skill management) are still hidden locally where needed.

## Pros and Cons of the Options

### Option A - Role checks inside each page and component

- ✅ Simple to start with, no shared abstraction
- ❌ Rules drift apart between sidebar, page and shortcuts
- ❌ No overview of who may open what
- ❌ Easy to forget a check when adding a page

### Option B - Roles from the Keycloak token

- ✅ No extra request, roles are available right after login
- ❌ The token can hold several roles, so the frontend would need its own priority logic
- ❌ Says nothing about project management, which lives in the backend database
- ❌ Risk of disagreeing with the backend's effective group

### Option C - Central route-based access policy

- ✅ One source of truth for all navigation surfaces
- ✅ Type-checked: every protected route must have a permission entry
- ✅ Project-manager scoping fits in as one extra rule
- ✅ Matches the backend because it uses the backend's effective group
- ❌ Route permissions have to be kept in sync with the backend by hand
- ❌ Covers routes, not single actions inside a page

### Option D - Fine-grained capabilities from the backend

- ✅ Very precise, one flag per action
- ✅ Frontend and backend cannot disagree
- ❌ Needs a new backend endpoint and contract for every capability
- ❌ Much larger change than the current requirements justify

## Consequences

**Positive:**
- Sidebar, router, dashboard widgets and shortcuts always agree on what a user can reach.
- A PM who is only a member of the selected project does not see the PM area and cannot open it by URL. This matches the backend, which would answer with 403.
- A new protected route cannot be added without deciding who may open it.

**Negative / Trade-offs:**
- The permission table exists twice, in the backend security config and in `accessPolicy.ts`, and has to be changed in both places.
- Only routes wrapped in a guard are blocked by URL. Other restricted routes are only hidden from navigation and rely on the backend to reject the requests.
- HR is grouped with ADMIN in the frontend, but some backend endpoints are ADMIN-only (for example `/api/v1/admin/projects` and skill management). The frontend handles these cases locally, so the group alone does not fully describe what HR can do.
- The profile, and with it the permission group and the onboarding flag, is loaded once per session. A role change takes effect after a reload.

**Follow-up actions:**
- [x] Central policy with `AppRoute`, `routePermissions` and `canAccessRoute` (`src/auth/accessPolicy.ts`)
- [x] Project-manager scoping (`MANAGER_ASSIGNMENT_ROUTES`) and `ManagerAreaGuard` in the router
- [ ] Wrap `/admin` in a role guard. It is currently only hidden in the sidebar, so a `USER` who opens the URL sees the page shell and gets 403 responses.
- [ ] Decide whether HR keeps the ADMIN-like route access or gets its own, narrower set (see the comment on `/hire-setup` in `accessPolicy.ts`).
- [ ] Enforce the one-project-per-user rule for regular users in the backend.

## Implementation Guidelines

### Where the inputs come from

| Input | Source | Meaning |
|---|---|---|
| `profile.permissionGroup` | `GET /api/v1/users/me`, loaded by `AuthProvider` | The user's effective group, chosen by the backend with priority `ADMIN > HR > PM > USER`. |
| `canManageSelected` | `ProjectProvider` (`useProjectContext()`) | `true` only when the current user is the assigned manager of the selected project. For PMs it comes from `GET /api/v1/projects/managed`. For admins it is true only for projects they personally manage. |
| `profile.hasCompletedOnboarding` | `GET /api/v1/users/me` | Users who finished onboarding can no longer open `/onboarding`. |

### Route matrix

| Route | USER | PM | HR | ADMIN | PM must manage the selected project |
|---|---|---|---|---|---|
| `/`, `/chat`, `/buddy`, `/board`, `/knowledge-base`, `/onboarding`, `/settings`, `/profile` | ✅ | ✅ | ✅ | ✅ | - |
| `/hire-setup` | | ✅ | ✅ | ✅ | no |
| `/blueprints`, `/data-ingestion` | | ✅ | ✅ | ✅ | yes |
| `/pm-dashboard`, `/team-management`, `/insights/faq`, `/insights/knowledge-gaps`, `/insights/knowledge-requests`, `/insights/onboarding` | | ✅ | ✅ | ✅ | yes |
| `/admin` | | | ✅ | ✅ | - |

`/arrival-steps` and `/starter-work` only redirect to `/hire-setup` and share its entry. The source of truth is always `accessPolicy.ts`. Update this table when it changes.

### Where the policy is applied

- **Router.** `AuthGuard` (`src/router/AuthGuard.tsx`) handles authentication, the login redirect, the skill assessment redirect to `/skill-wizard` and the onboarding block for completed users. `ManagerAreaGuard` (`src/router/AppRouter.tsx`) wraps the PM area, data ingestion, blueprints and hire setup, and redirects to `getDefaultRoute` when `canAccessRoute` fails. It waits for the project context to load, so a managing PM is not bounced during the initial load.
- **Navigation.** `SideBar`, the dashboard widget catalog, `useGlobalShortcuts` and the keyboard shortcut overview call `canAccessRoute` with `canManageSelected` to decide what to show.
- **Inside pages.** Actions that are narrower than their page (for example the ADMIN-only skills tab in `AdminPage`) are hidden in the page itself.

### Adding a new protected route

1. Add the path to the `AppRoute` union in `src/auth/accessPolicy.ts`.
2. Add its allowed groups to `routePermissions`. The code does not compile until this is done.
3. If the page shows data of the selected project and PMs should only reach it for projects they manage, add it to `MANAGER_ASSIGNMENT_ROUTES`.
4. If it has dynamic segments (like `/team/:userId`), add a prefix to `routePrefixes` so the real URL maps back to the route.
5. Wrap the route in `ManagerAreaGuard` in `AppRouter.tsx` if it must not be reachable by URL for users who lack access.
6. Make sure the backend enforces the same rule. The frontend policy only decides visibility.

## Links

- [ADR Index](index.md)
- [ADR-015: Use Keycloak for Authentication and Authorization](adr-015-use-keycloak-authentication-authorization.md)
- [sprintstart-frontend](https://github.com/SprintStartProject/sprintstart-frontend)
