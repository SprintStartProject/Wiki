# ADR-010: Secrets Management Strategy

| Field        | Value          |
|---|---|
| **Status**       | `Accepted`     |
| **Date**         | 2026-05-19     |
| **Deciders**     | *Full Team*    |
| **Supersedes**   | –              |
| **Superseded by**| –              |

---

## Context

The system consists of multiple repositories and components (Spring Boot backend, AI/Retrieval
service, and frontend) deployed via GitHub Actions CI/CD pipelines. Each component requires
sensitive configuration values such as:

- Database credentials
- JWT signing secrets
- External API keys
- Service-to-service authentication tokens

This ADR defines the secrets management strategy **at the system level**, applying to all
repositories. Per-service implementation details (e.g. how each stack reads environment variables)
are documented in service-specific READMEs or ADRs.

Exposing sensitive values in source control introduces a high-risk security surface, including
credential leakage via Git history, accidental exposure in pull requests, and CI log artifacts
containing secrets. A consistent, enforceable strategy is required across local development,
CI/CD pipelines, and production environments.

## Decision Drivers

- No credentials must ever be stored in any repository
- Strategy must apply uniformly across all services regardless of tech stack
- Developer onboarding must remain straightforward
- Must be compatible with GitHub Actions-based CI/CD without additional infrastructure

## Considered Options

- **Option A** – GitHub Secrets + `.env` (untracked) + `.env.template` (tracked)
- **Option B** – Secrets stored in `application.yml` / config files per service
- **Option C** – Dedicated secret manager (HashiCorp Vault / AWS Secrets Manager)

## Decision

**Chosen option: Option A** – A hybrid approach using GitHub Secrets for CI/CD injection,
untracked local `.env` files for development, and a tracked `.env.template` as a configuration
contract shared across all repositories.

## Rationale

Option A provides the right balance of security and operational simplicity for the current
stage. GitHub Secrets are scoped per environment, injected only at workflow runtime, and never
persisted in build artifacts. The `.env` + `.env.template` pattern ensures local development
remains straightforward while the template file acts as a living contract that keeps all
developers and services aligned on required configuration. No additional infrastructure is
required.

Each service reads environment variables in an idiomatic way for its stack:

| Service       | Stack               | Env Resolution                          |
|---|---|---|
| Backend       | Spring Boot / Kotlin | `application.yml` variable placeholders |
| AI/Retrieval  | Python              | `os.environ` / `python-dotenv`          |
| Frontend      | React / TypeScript  | Build-time `VITE_*` / runtime config    |

## Pros and Cons of the Options

### Option A – GitHub Secrets + `.env` + `.env.template`

- ✅ No credentials stored in any repository
- ✅ Clear separation between local, CI, and production environments
- ✅ `.env.template` improves onboarding and acts as a cross-repo configuration contract
- ✅ Compatible with GitHub Actions without additional infrastructure
- ✅ Stack-agnostic — applies uniformly regardless of service language or framework
- ❌ Developers must manually manage local `.env` files
- ❌ Risk of local environments drifting if `.env` diverges from `.env.template`
- ❌ No centralised secret rotation mechanism at this stage

### Option B – Secrets in config files (`application.yml` / `.env` committed)

- ✅ Zero setup friction for developers
- ❌ High risk of credential leakage via Git history
- ❌ Violates baseline security practices
- ❌ Not compatible with CI/CD security requirements

### Option C – Dedicated Secret Manager (Vault / AWS Secrets Manager)

- ✅ Centralised rotation, auditing, and access control
- ✅ Strong production-grade secret lifecycle management
- ❌ Significant operational overhead and infrastructure complexity
- ❌ Requires deployment environment maturity not present at this stage
- ❌ Not justified for the current team size and project phase

## Consequences

**Positive:**
- No sensitive credentials are stored in version control across any repository
- Onboarding is straightforward via `.env.template` in every repo
- CI/CD pipelines remain simple — no external secret manager dependency

**Negative / Trade-offs:**
- No centralised secret rotation — secrets must be updated manually per GitHub environment
- Developer discipline required to avoid logging environment variables locally or in CI

**Follow-up actions:**
- Every repository must include `.env.template` and a `.gitignore` entry blocking `.env`
- CI pipelines must validate presence of required secrets before deployment steps
- Logging configurations in all services must redact environment variables

## Links
