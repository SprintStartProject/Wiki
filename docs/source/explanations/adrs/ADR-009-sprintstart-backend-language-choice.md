# ADR-009: Kotlin as Primary Language

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-12 |
| **Deciders** | *Full Team* |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

We need to choose a primary JVM language for our Spring Modulith backend (ADR-002). The choice
affects day-to-day developer productivity, code expressiveness, safety guarantees, and the
long-term maintainability of the codebase. The team has existing experience on the JVM and the
system is a backend service without UI concerns.

## Decision Drivers

- Null-safety as a language-level guarantee reduces an entire class of runtime errors
- Concise, expressive syntax reduces boilerplate in domain and data classes
- Full interoperability with the Java/Spring ecosystem
- Coroutine support for non-blocking I/O without callback complexity

## Considered Options

- **Option A** – Kotlin
- **Option B** – Java

## Decision

**Chosen option: Option A** – Kotlin is used as the primary language across all modules.
Java interoperability is maintained, but all new code is written in Kotlin.

## Rationale

The team already has Java background, so the step up to Kotlin is a natural one. 
Kotlin's null-safety system catches potential null pointer
exceptions at compile time rather than at runtime, which is particularly valuable in a backend
service where data flows through multiple layers. Data classes significantly reduce boilerplate
for model and event classes — a daily concern given our module structure (ADR-004).
Coroutines provide structured, readable non-blocking I/O without the complexity of reactive
streams, and integrate well with Spring WebFlux and Spring Data if needed in the future.
Spring has first-class Kotlin support, including Kotlin-specific DSLs and extension functions.

## Pros and Cons of the Options

### Option A – Kotlin

- ✅ Null-safety at compile time — eliminates a class of NullPointerException bugs
- ✅ Data classes reduce boilerplate for model and event classes
- ✅ Coroutines enable non-blocking I/O with sequential, readable code
- ✅ Full Java interoperability — all Spring and Java libraries work without wrappers
- ✅ First-class Spring support including Kotlin DSLs and extension functions
- ❌ Slightly longer compile times than Java in some configurations
- ❌ Slight adoption barrier at first

### Option B – Java

- ✅ Team experience in Java
- ✅ Slightly faster compile times in some toolchain setups
- ❌ Verbose boilerplate for data classes, builders, and null checks (partially mitigated by
  Records and newer Java features, but still less concise than Kotlin)
- ❌ Null safety requires external tooling (e.g. `@NonNull` annotations) rather than being
  built into the type system
- ❌ No native coroutine support — reactive programming requires a steeper learning curve

## Consequences

**Positive:**
- Concise data and event classes across all modules (aligns with ADR-0003 module structure)
- Compile-time null-safety reduces defensive null checks throughout the codebase
- Coroutines available if non-blocking I/O becomes a requirement without introducing a
  reactive framework

**Negative / Trade-offs:**
- All contributors must be comfortable with Kotlin; onboarding Java-only developers requires
  a short ramp-up period
- Kotlin compiler plugins (e.g. for Spring, JPA) must be configured correctly — misconfiguration
  can cause subtle runtime issues (e.g. open classes for JPA entities)

**Follow-up actions:**
- Ensure Kotlin compiler plugins for Spring (`kotlin-spring`) and JPA (`kotlin-jpa`) are
  configured in the build file
- Add a Kotlin style guide reference to the team wiki (e.g. official Kotlin coding conventions)
- Agree on coroutine usage boundaries — whether coroutines are used broadly or only where
  explicitly needed

## Links

- [ADR Index](adr-index.md)
- [ADR-006: Spring Modulith as Application Framework](ADR-006-sprintstart-backend-architecture.md)
- [Kotlin + Spring Boot Documentation](https://docs.spring.io/spring-framework/reference/languages/kotlin.html)
- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
