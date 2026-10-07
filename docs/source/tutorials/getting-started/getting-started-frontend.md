# sprintstart-frontend

## Setup Guide

### Prerequisites

- [Node.js](https://nodejs.org/) 20.19+ or 22.12+ (the minimum Vite 8 accepts). CI and the Docker image use Node 24, which is the safe choice.
- npm
- A running backend and Keycloak (see [Backend and Keycloak](#backend-and-keycloak) below)

> **Note on newer Node versions:** on Node 25 or newer, Node's built-in `localStorage` hides the one jsdom provides and a large part of the unit tests fail. Run the tests with the built-in one switched off:
>
> ```bash
> NODE_OPTIONS=--no-experimental-webstorage npm run test
> ```

### Installation

1. Clone the repository and navigate into the project folder:
   ```bash
   git clone https://github.com/SprintStartProject/sprintstart-frontend
   cd sprintstart-frontend
   ```
2. Install the project dependencies. This also runs `keycloakify sync-extensions`, which copies the Keycloak login theme pages into `src/keycloak-theme/`:
   ```bash
   npm install
   ```

### Environment Variables

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

| Variable | Default | Purpose |
|---|---|---|
| `VITE_KEYCLOAK_CLIENT_ID` | `sprintstart-frontend` | Keycloak client ID in the `sprintstart` realm. |
| `VITE_KC_DEV` | unset | Set to `true` only for the Keycloak login theme preview (`npm run dev-keycloak-theme`). |

No API or Keycloak URLs have to be configured. The frontend sends all requests to its own origin and they are forwarded from there.

(backend-and-keycloak)=
### Backend and Keycloak

The frontend does not work on its own. Login goes through Keycloak and every page loads its data from the backend. The dev server forwards:

| Path | Target |
|---|---|
| `/api`, `/v1` | Backend on `http://127.0.0.1:8080` |
| `/auth` | Keycloak on `http://127.0.0.1:8081` |

The easiest way to get both is the `docker-compose.yaml` in `sprintstart-backend`, which starts the databases, Keycloak and the backend. See [getting-started-backend](getting-started-backend.md) for running the backend locally instead (Keycloak is still needed for that route). How to create test users and assign roles in Keycloak is described in the frontend `README.md` under "Authentication & User Setup".

### Development

To start the local development server, run:

```bash
npm run dev
```

The application will be accessible in your browser at: **http://localhost:5173/**

### Frontend via Docker

To build and serve only the frontend through nginx via docker compose, run:

```bash
docker compose up --build
```

The application will be accessible in your browser at: **http://localhost:3000/**. Backend and Keycloak still have to run on the host, nginx forwards to them through `host.docker.internal`.

### Definition of Done

Before opening a pull request, run:

```bash
npm run try
```

This runs install, format check, build, lint, unit tests and accessibility tests in one go. All steps have to pass.

### Further Reading

The frontend repository contains the detailed developer documentation:

- `README.md`: setup, scripts and developer notes
- `docs/FRONTEND_ARCHITECTURE.md`: architecture, routing, state and API layer
- `docs/FRONTEND_CODING_STANDARDS.md`: TypeScript, React, Tailwind and accessibility rules
- `docs/FRONTEND_DOCUMENTATION_GUIDELINES.md`: TSDoc and comment rules
- `docs/testing_strategy.md`: Vitest, MSW and vitest-axe setup
- `docs/UI_DESIGN_DECISIONS.md`: why the UI rules exist, and the open UI consistency items
- `AGENTS.md`: short entry point for humans and AI agents, with pointers to the rules above
