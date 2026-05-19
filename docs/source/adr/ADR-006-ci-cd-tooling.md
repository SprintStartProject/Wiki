# ADR-006: CI/CD Pipeline

| Field         | Value       |
|---|---|
| **Status**        | `Accepted`  |
| **Date**          | 2026-05-19  |
| **Deciders**      | *Full Team* |
| **Supersedes**    | –           |
| **Superseded by** | –           |

---

## Context

The project requires a unified CI/CD and observability strategy across all teams and repositories
(backend, frontend, AI/Retrieval services). Docker image building is already implemented. The
pipeline must enforce quality gates from Sprint 0 while remaining lightweight for a small team.

## Decision Drivers

- Quality gates must be enforced automatically on every pull request across all repos
- Observability must cover all services from a single shared platform
- Operational overhead must stay low for a small team
- Docker is already in use — CI/CD must build on top of it

## Considered Options

- **Option A** – GitHub Actions + Docker image artifacts + Prometheus/Grafana
- **Option B** – Jenkins or self-hosted CI runner
- **Option C** – No formal pipeline — manual build and deploy

## Decision

**Chosen option: Option A** – GitHub Actions orchestrates the full pipeline across all
repositories, Docker images are treated as immutable build artifacts, and Prometheus + Grafana
provide a shared observability platform for all services.

## Rationale

GitHub Actions integrates natively with all repositories and requires no additional
infrastructure. Docker images built in CI are versioned and consumed directly by CD without
rebuilding, ensuring environment consistency across teams. Prometheus and Grafana form a
shared observability platform covering backend JVM metrics, frontend performance, AI inference
latency, and container health via Spring Boot Actuator + Micrometer. SonarQube is deferred as
it overlaps significantly with `ktlint` and `detekt` at the current project scale.

## Pros and Cons of the Options

### Option A – GitHub Actions + Docker + Prometheus/Grafana

- ✅ Native GitHub integration across all repositories — no additional infrastructure
- ✅ Immutable Docker artifacts ensure reproducible, consistent deployments
- ✅ Shared observability platform covers backend, frontend, AI, and containers
- ✅ `ktlint` + `detekt` enforce code quality without external tooling overhead
- ❌ Docker image versioning requires team discipline
- ❌ Prometheus/Grafana require initial setup overhead

### Option B – Jenkins or self-hosted CI runner

- ✅ More control over runner environment
- ❌ Significant operational overhead to maintain
- ❌ Not justified for current team size and project scope

### Option C – No formal pipeline

- ✅ Zero setup effort
- ❌ No quality gates — defects reach the server unchecked
- ❌ Manual deployments are error-prone and not reproducible

## Consequences

**Positive:**
- Every PR across all repositories is validated through linting, static analysis, tests, and a Docker build
- CI and CD are cleanly separated — CD only pulls and runs pre-built images
- Single observability platform shared across all teams from Sprint 0

**Negative / Trade-offs:**
- SonarQube quality dashboard not available at this stage
- Prometheus/Grafana require initial configuration before providing value
