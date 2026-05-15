# ADR-002: The backend architecture

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-10 |
| **Deciders** | Full backend team |
| **Supersedes** | - |
| **Superseded by** | - |

---

## Context

We are building an internal backend service with clearly separable business domains.
The team is small, which limits our capacity to operate and maintain distributed infrastructure.
At the same time, the system is expected to grow, so the architecture should preserve the option
to extract individual domains into independent services later without a costly rewrite.

## Decision Drivers

- Small team — low capacity for distributed system operations
- Business logic is more or less separable into distinct domains
- Single deployable unit preferred for operational simplicity
- Future extraction into microservices must remain feasible
- Kotlin works well with the Java background of the team, while offering a more modern language

## Considered Options

- Option A – Spring Modulith with Kotlin
- Option B – Classic Microservices
- Option C – Plain Monolith (without Modulith)

## Decision

Chosen option: Option A – Spring Modulith enforces explicit module boundaries within a single
deployable unit, matching our need for domain separation without the operational overhead of
distributed services.

## Rationale

Spring Modulith gives us the structural discipline of microservices — explicit module boundaries,
enforced by tooling — while keeping the operational simplicity of a monolith. Inter-module
communication via Spring Application Events means module boundaries are explicit in code and
verifiable at test time. Kotlin integrates naturally with the Spring ecosystem and reduces
boilerplate through null-safety, data classes, and coroutine support. Should individual modules
need to scale independently in the future, the event-based boundaries make extraction into
separate services a realistic path. More on the implementation in [ADR-003](./ADR-003-internal-backend-architecture.md).

## Pros and Cons of the Options

### Option A – Spring Modulith with Kotlin

- ✅ Single deployable unit — simple CI/CD, monitoring, and local development
- ✅ Module boundaries enforced by @ApplicationModuleTest at test time
- ✅ No network overhead — inter-module calls happen in-process
- ✅ Migration path preserved — modules can be extracted into services later
- ✅ Kotlin reduces boilerplate and eliminates a class of null-related bugs
- ❌ Shared deployment — a fault in one module affects the entire service
- ❌ Scaling granularity — the whole service must scale together
- ❌ Spring Modulith is relatively young; some edge cases may need workarounds

### Option B – Classic Microservices

- ✅ Independent deployability and scaling per service
- ✅ Strong isolation — a failure in one service does not affect others
- ❌ Network latency and overhead for every inter-service call
- ❌ Distributed tracing, service discovery, and independent deployments require
significant operational capacity a small team cannot absorb
- ❌ Data consistency across service boundaries adds substantial engineering effort

### Option C - Plain Monolith

- ✅ Simplest possible setup — no framework conventions to learn
- ❌ No tooling enforcement of module boundaries — they erode over time
- ❌ A big-ball-of-mud monolith would make future extraction into services very costly

## Consequences

**Positive:**
- Developers work in a single codebase with a clear, consistent module structure
- No infrastructure needed for inter-module communication
- Spring Modulith's verification catches illegal cross-module dependencies early

**Negative / Trade-offs:**
- Modules share a database; schema-level isolation must be enforced by convention
- Horizontal scaling applies to the entire service, not individual modules
- Team needs to internalize Spring Modulith conventions (especially event-based communication)

**Follow-up actions:**
- Establish schema isolation strategy (separate DB schemas per module)
- Add @ApplicationModuleTest to the standard test setup for every module

## Links

- [ADR Index](adr-index.md)
- [ADR-003](./ADR-003-internal-backend-architecture.md)
- [Spring Boot with Kotlin](https://docs.spring.io/spring-framework/reference/languages/kotlin.html)
- [Spring Modulith](https://docs.spring.io/spring-modulith/reference/)
