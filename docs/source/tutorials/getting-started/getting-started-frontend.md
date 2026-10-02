# sprintstart-frontend

## Setup Guide

### Prerequisites

- [Node.js](https://nodejs.org/) 24 (the version used by CI and the Docker image). Vite 8 needs at least Node 20.19 or 22.12.
- npm
- A running backend and Keycloak (see [Backend and Keycloak](#backend-and-keycloak) below)

> **Note on newer Node versions:** with Node 26, `localStorage` is not available in the Vitest environment and a large part of the unit tests fail. Use Node 24, or run the tests with `NODE_OPTIONS="--localstorage-file=<some temp file>"`.

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

### Backend and Keycloak

The frontend does not work on its own. Login goes through Keycloak and every page loads its data from the backend. The dev server forwards:

| Path | Target |
|---|---|
| `/api`, `/v1` | Backend on `http://127.0.0.1:8080` |
| `/auth` | Keycloak on `http://127.0.0.1:8081` |

The easiest way to get both is the `docker-compose.yaml` in `sprintstart-backend`, which starts the databases, Keycloak and the backend. See [getting-started-backend](getting-started-backend.md) for running the backend locally instead. How to create test users and assign roles in Keycloak is described in the frontend `README.md` under "Authentication & User Setup".

### Development

To start the local development server, run:

```bash
npm run dev
```

The application will be accessible in your browser at: **http://localhost:5173/**

### One Command Start

To build and serve the frontend through nginx via docker compose, run:

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
- `docs/testing_strategy.md`: Vitest, MSW and vitest-axe setup
