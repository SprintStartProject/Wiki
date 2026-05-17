# ADR-003: Event-Based Inter-Module Communication

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-12 |
| **Deciders** | *Full Team* |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

Within our Spring Modulith application (ADR-002), modules need a defined and consistent way to
interact with each other. Without a clear rule, developers will default to direct Spring Bean
injection across module boundaries, which silently couples modules and undermines the modularity
we chose Spring Modulith for.

## Decision Drivers

- Inter-module coupling must be explicit, auditable, and enforceable
- Communication pattern should align naturally with Spring Modulith's model
- The chosen pattern should not block a future extraction of modules into separate services
- Simplicity preferred — low boilerplate for the common case

## Considered Options

- **Option A** – Spring Application Events (`ApplicationEventPublisher` / `@ApplicationModuleListener`)
- **Option B** – Direct Spring Bean injection across module boundaries
- **Option C** – REST / HTTP calls between modules (in-process or extracted)

## Decision

**Chosen option: Option A** – Modules communicate exclusively through Spring Application Events.
No module may directly inject or call a Spring Bean owned by another module.

## Rationale

Spring Application Events are the communication primitive Spring Modulith is designed around.
Publishing a named event from `api/` makes the contract between modules explicit in code:
any developer can open a module's `api/` package and immediately understand what that module
broadcasts to the rest of the system. `@ApplicationModuleListener` is used instead of
`@EventListener` so that Spring Modulith can verify, document, and trace all cross-module
interactions. This pattern also maps naturally to a message broker (e.g. Kafka, RabbitMQ)
if a module is ever extracted into a separate service — the event structure stays the same,
only the transport changes.

**Convention on sync vs. async:**
Spring's default event dispatch is synchronous. For events where the consumer must not run
within the publisher's transaction, `@ApplicationModuleListener(async = true)` or Spring
Modulith's transactional event publication must be used explicitly. This decision must be made
consciously per listener and documented inline.

## Pros and Cons of the Options

### Option A – Spring Application Events

- ✅ Loose coupling — the publishing module has no compile-time dependency on consumers
- ✅ Explicit contracts — every cross-module interaction is represented by a named event class
- ✅ Verifiable — Spring Modulith documents and verifies event flows via `@ApplicationModuleTest`
- ✅ Migration-ready — event structure maps directly to a message broker if needed
- ❌ Synchronous by default — async behavior requires explicit opt-in per listener
- ❌ Slightly more indirection than a direct method call — debugging requires following events

### Option B – Direct Spring Bean Injection Across Modules

- ✅ Familiar and straightforward — no new pattern to learn
- ❌ Creates invisible compile-time coupling between modules
- ❌ Violates Spring Modulith's module boundary model — caught as a test failure
- ❌ Makes future service extraction significantly harder

### Option C – In-Process REST / HTTP Calls

- ✅ Mirrors what inter-service communication would look like after extraction
- ❌ Unnecessary network stack overhead for in-process calls
- ❌ High boilerplate for a pattern that offers no benefit within a single JVM

## Consequences

**Positive:**
- Every cross-module interaction is traceable to a named event class in a module's `api/` package
- `@ApplicationModuleTest` verifies that no module directly accesses another module's internals
- The event model is a direct stepping stone to async messaging if modules are ever extracted

**Negative / Trade-offs:**
- Synchronous event dispatch (the default) provides structural decoupling but not runtime
  decoupling — publishers and consumers still run in the same thread and transaction unless
  explicitly configured otherwise
- Developers unfamiliar with event-driven patterns may find debugging less straightforward
  than following a direct call stack

**Follow-up actions:**
- Establish and document the team's rule for when to use async vs. synchronous event dispatch
- Add event flow verification to the `@ApplicationModuleTest` setup for every module
- Define where event classes live within each module → ADR-004

## Links

- [ADR Index](adr-index.md)
- [ADR-002: Spring Modulith as Application Framework](ADR-002-spring-modulith.md)
- [ADR-004: Backend Module Structure](ADR-004-module-structure.md)
- [Spring Modulith — Application Events](https://docs.spring.io/spring-modulith/reference/events.html)
