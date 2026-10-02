# ADR-019: Custom Keycloak Login Theme with Keycloakify

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-06-19 |
| **Deciders** | Frontend Team |
| **Supersedes** | - |
| **Superseded by** | - |

---

## Context

SprintStart uses Keycloak for authentication ([ADR-015](adr-015-use-keycloak-authentication-authorization.md)). Login, registration, password reset and the other account flows are not pages of the SPA. Keycloak renders them itself, using the login theme configured for the realm.

Keycloak's default login theme looks nothing like the SprintStart app. New hires meet the login page before anything else, so a generic Keycloak screen followed by a completely different app makes a poor first impression. The login and registration pages should use the same colors, typography and light and dark mode as the rest of the app.

Keycloak themes are traditionally written as FreeMarker (`.ftl`) templates with their own CSS. The frontend team works in React, TypeScript and Tailwind ([ADR-011](adr-011-frontend-framework.md)), and the app's design tokens live in `src/styles/index.css`.

The account console (where users manage their own Keycloak account) is not part of the SprintStart user flow. Profile settings are handled inside the app.

## Decision Drivers

- Login and registration look and feel like the rest of the app, including light and dark mode.
- The theme is written with the same stack and design tokens as the app, so the frontend team can maintain it.
- Keycloak keeps handling credentials. The SPA never sees a password.
- Pages can be developed and previewed without a running Keycloak.
- Updates of Keycloak's own pages should not require rewriting the theme.

## Considered Options

- **Option A** - Default Keycloak theme, adjusted only through theme properties, logo and CSS
- **Option B** - Hand-written FreeMarker theme
- **Option C** - Login form inside the SPA using the password grant (Direct Access Grants)
- **Option D** - React theme built with Keycloakify inside the frontend repository

## Decision

**Chosen option: Option D** - The login theme is built with Keycloakify 11 and `@keycloakify/login-ui` inside `sprintstart-frontend`, so it is written in React with the app's own design tokens and packaged as a Keycloak theme JAR.

## Rationale

Keycloakify lets the team write Keycloak pages as React components. The theme is built from the same Vite project and the same `index.html` as the app and imports the app's `index.css`. It therefore uses the same tokens and the same light and dark mode. Keycloak still renders the page and processes the form, so credentials never pass through the SPA.

`@keycloakify/login-ui` provides a React version of every Keycloak login page. The team only takes ownership of the files it actually changes (`npx keycloakify own`). All other pages are copied in by the `postinstall` script (`keycloakify sync-extensions`) and stay in line with the library version. This keeps the amount of code the team maintains small and makes Keycloak updates cheaper than with a fully hand-written theme.

Adjusting the default theme (Option A) cannot get close enough to the app's design without effectively rewriting it. A FreeMarker theme (Option B) would mean a second template language and a second copy of the design system. A login form inside the SPA (Option C) would require the password grant, which is deprecated in OAuth 2.1, gives the SPA direct access to passwords and loses Keycloak features such as passkeys, OTP and identity provider login.

## Pros and Cons of the Options

### Option A - Default theme with properties and CSS

- ✅ No build step, nothing to maintain beyond a few files
- ✅ Keycloak updates do not break anything
- ❌ Layout and components stay Keycloak's, only colors and logo change
- ❌ Hard to match the app's design and dark mode

### Option B - Hand-written FreeMarker theme

- ✅ Full control over every page
- ✅ No extra library
- ❌ Second template language that nobody in the frontend team uses
- ❌ Design tokens and components have to be duplicated in plain CSS
- ❌ Every Keycloak page has to be written and kept up to date by hand

### Option C - Login form inside the SPA

- ✅ Login looks exactly like the app, no separate theme at all
- ❌ Requires the password grant, which is deprecated and not recommended for public clients
- ❌ The SPA handles passwords directly
- ❌ Loses Keycloak flows such as passkeys, OTP, email verification and identity provider login

### Option D - Keycloakify React theme

- ✅ Same stack, same design tokens and same components as the app
- ✅ Only customized files are owned, the rest follows the library
- ✅ Local preview with mock data and Storybook stories, no Keycloak needed
- ✅ Keycloak keeps full control of credentials and auth flows
- ❌ Extra build step that needs Java and Maven to produce the JAR
- ❌ The built JAR has to be brought into the Keycloak image separately from the frontend deployment
- ❌ Tied to the Keycloakify and `@keycloakify/login-ui` release cycle

## Consequences

**Positive:**
- Login, registration and password reset match the app, including light and dark mode.
- The theme is maintained by the frontend team in the frontend repository with the usual tools.
- Most Keycloak pages need no code of their own, only the owned files differ from the library.

**Negative / Trade-offs:**
- The theme is not deployed with the frontend. The JAR is built by hand with `npm run build-keycloak-theme` and committed to `sprintstart-backend/infra/keycloak/themes/`, from where the Keycloak Docker image picks it up. Nothing checks that this JAR matches the current frontend code. The JAR was last updated on 2026-08-13, and theme files in the frontend changed after that.
- The login page runs on the same origin as the app (`/auth` is proxied by Vite and nginx), so it can read the app's `localStorage` settings such as the selected light or dark theme. This only works as long as both stay on the same origin.
- Because the theme is built from the app's `index.html`, it also inherits the boot splash, which `src/main.tsx` has to dismiss right away for the login page.

**Follow-up actions:**
- [x] Initialize Keycloakify and adapt the login and registration pages to the app design
- [x] Configure `loginTheme: "sprintstart-frontend"` in the realm imports (local and Kubernetes)
- [ ] Build the theme JAR in CI and hand it to the Keycloak image automatically instead of copying it by hand
- [ ] Rebuild and update the JAR in `sprintstart-backend` so it matches the current theme code

## Implementation Guidelines

### Structure

| Path | Purpose |
|---|---|
| `src/main.tsx` | Single entry. Loads the theme when Keycloak provides `window.kcContext`, the theme dev preview when `VITE_KC_DEV=true`, and the app otherwise. |
| `src/keycloak-theme/kc.gen.tsx` | Generated by Keycloakify. Do not edit by hand. |
| `src/keycloak-theme/login/` | The login theme. Only files that are tracked in Git are owned and customized. |
| `src/keycloak-theme/login/styleLevelCustomization.tsx` | Imports the app's `index.css` and the theme's `login.css`, in that order. |
| `src/keycloak-theme/login/components/Template/Template.tsx` | The shared page frame for all login pages. Most visual changes belong here. |

All other files under `src/keycloak-theme/login/` are copied in from `@keycloakify/login-ui` by `npm install` and are ignored by Git (`src/keycloak-theme/.gitignore`). Changes to them are lost on the next install.

### Customizing a page

1. Check whether the change fits into `Template.tsx` or `login.css`. That covers most cases.
2. If a single page needs its own markup, take ownership of it first: `npx keycloakify own --path "login/pages/<page>/Page.tsx"`. The file is then tracked in Git.
3. Use the app's design tokens (`bg-app-*`, `text-app-*`) as everywhere else in the frontend.

### Developing and previewing

- `npm run dev-keycloak-theme` starts Vite with `VITE_KC_DEV=true` and renders the theme with mock data, without Keycloak.
- `npm run storybook` shows the theme pages as stories (`Page.stories.tsx`).

### Building and deploying

1. `npm run build-keycloak-theme` builds the app and then the theme. It needs Java and Maven and writes the JAR to `dist_keycloak/` (ignored by Git).
2. Copy the JAR to `sprintstart-backend/infra/keycloak/themes/`. The Keycloak `Dockerfile` copies every JAR from there into `/opt/keycloak/providers/`.
3. The realm imports set `loginTheme` to `sprintstart-frontend`, which is the theme name Keycloakify derives from the package name.

The account theme is disabled (`accountThemeImplementation: "none"` in `vite.config.ts`), so the Keycloak account console keeps its default look.

## Links

- [ADR Index](index.md)
- [ADR-011: Frontend Framework](adr-011-frontend-framework.md)
- [ADR-015: Use Keycloak for Authentication and Authorization](adr-015-use-keycloak-authentication-authorization.md)
- [Keycloakify documentation](https://docs.keycloakify.dev)
- [sprintstart-frontend](https://github.com/SprintStartProject/sprintstart-frontend)
