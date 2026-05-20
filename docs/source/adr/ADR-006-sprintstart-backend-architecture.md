# ADR-002: Spring Modulith as Application Framework

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-12 |
| **Deciders** | *Full Team* |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

We are building an internal backend service with clearly separable business domains. We need a
framework that supports modular decomposition within a single deployable unit, enforces module
boundaries through tooling rather than convention alone, and integrates well with our chosen
language (ADR-005) and communication strategy (ADR-003).

## Decision Drivers

- Module boundaries must be explicit and verifiable, not just documented
- Single deployable unit preferred to keep operational complexity low for a small team
- Future extraction of individual modules into independent services must remain feasible
- Strong existing Spring Boot knowledge in the team

## Considered Options

- **Option A** – Spring Modulith
- **Option B** – Classic Microservices
- **Option C** – Plain Spring Boot Monolith

## Decision

**Chosen option: Option A** – Spring Modulith provides tooling-enforced module boundaries within
a single deployable unit, which matches our need for domain separation without the operational
overhead of a distributed system.

## Rationale

Spring Modulith builds on top of Spring Boot and adds explicit, verifiable module boundaries via
package conventions and `@ApplicationModuleTest`. It treats modules as first-class citizens:
their public API is defined by what is visible in the `api/` package, and cross-module access
to internals is caught at test time. This gives us the structural discipline we want without
requiring a distributed system. It also integrates naturally with Spring Application Events
(ADR-0003), which is our chosen inter-module communication mechanism.

## Pros and Cons of the Options

### Option A – Spring Modulith

- ✅ Tooling-enforced module boundaries — violations caught by `@ApplicationModuleTest`
- ✅ Single deployable unit — simple CI/CD, monitoring, and local development
- ✅ Built-in support for event-based inter-module communication
- ✅ Familiar Spring Boot foundation — minimal learning curve for the team
- ✅ Migration path preserved — clean module boundaries enable future service extraction
- ❌ Relatively young framework — some edge cases may require workarounds
- ❌ Shared deployment — a fault in one module can affect the whole service through the shared database

### Option B – Classic Microservices

- ✅ Independent deployability and scaling per service
- ✅ Strong fault isolation between services
- ❌ Network overhead and latency for every inter-service call
- ❌ Requires distributed tracing, service discovery, and independent pipelines —
  operational complexity a small team cannot absorb
- ❌ Data consistency across service boundaries requires significant additional engineering

### Option C – Plain Spring Boot Monolith

- ✅ Simplest possible setup — no additional conventions to learn
- ❌ No tooling enforcement of module boundaries — they erode silently over time
- ❌ A monolith without boundaries makes future extraction into services very costly

## Consequences

**Positive:**
- Module boundaries are part of the build and test pipeline, not just documentation
- The team works on a familiar Spring Boot foundation with a well-defined extension

**Negative / Trade-offs:**
- The entire service must be scaled as one unit
- Spring Modulith's relative immaturity means the team should follow release notes closely

**Follow-up actions:**
- Add `@ApplicationModuleTest` to the standard test setup for every module
- Define inter-module communication conventions → ADR-003
- Define internal module structure → ADR-004

## Links

- [ADR Index](adr-index.md)
- [ADR-003: Event-Based Inter-Module Communication](ADR-007-sprintstart-backend-event-based-communication.md)
- [ADR-004: Backend Module Structure](ADR-008-sprintstart-backend-module-structure.md)
- [ADR-005: Kotlin as Primary Language](ADR-009-sprintstart-backend-language-choice.md)
- [Spring Modulith Reference Documentation](https://docs.spring.io/spring-modulith/reference/)
