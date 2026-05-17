# ADR-004: Backend Module Structure

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-12 |
| **Deciders** | *Full Team* |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

With Spring Modulith (ADR-002) and event-based communication (ADR-003) in place, we need
a consistent internal structure for every module. Without a defined layout, developers will
organise packages differently across modules, making the codebase harder to navigate as it
grows. The structure must make each module's public surface, entry points, and external
dependencies immediately visible to any developer.

## Decision Drivers

- Any developer should be able to navigate an unfamiliar module without guidance
- The structure must clearly separate what is public (usable by other modules) from what is internal
- Adding a new module should have no ambiguity about where things go
- The layout should reflect the event-based communication model from ADR-003

## Considered Options

- **Option A** – Custom pragmatic structure (domain-first, layer-second)
- **Option B** – Hexagonal Architecture (Ports & Adapters)
- **Option C** – Classic 3-Layer Architecture (global Presentation / Business / Data packages)

## Decision

**Chosen option: Option A** – Each module follows a consistent package layout with an `api/` package,
`controller/`, `service/`, `repository/`, which hold the classic 3 layers of logic, but separated by logic, 
a `model/` package, and an `EventListener` class.

## Rationale

We organise by module (domain) first, and by layer second. Everything belonging to a domain
lives together. The `api/` package is the only part of a module that is visible to the rest of
the application — it contains exclusively the events this module publishes (ADR-003). All
other packages are internal. A `EventListener` class per module handles all
incoming events, making each module's dependencies on the rest of the system visible in one
place. This structure is intentionally low-ceremony: it does not require ports, adapters, or
use-case classes, and maps directly to what most Spring Boot developers already know, therefore
keeping a clean split of logic while not introducing too much boilerplate.

**Standard module layout:**

```
com.example.app
└── order/
    ├── api/                        ← PUBLIC: events this module publishes
    │   ├── OrderPlacedEvent.kt
    │   └── OrderCancelledEvent.kt
    ├── controller/                 ← INTERNAL: HTTP entry points, input validation
    │   └── OrderController.kt
    ├── service/                    ← INTERNAL: business logic, event publishing
    │   └── OrderService.kt
    ├── repository/                 ← INTERNAL: data access only
    │   └── OrderRepository.kt
    ├── model/                      ← INTERNAL: domain classes and DTOs
    │   ├── Order.kt
    │   └── OrderRequest.kt
    └── OrderEventListener.kt       ← INTERNAL: handles all incoming events from other modules
```

**Layer responsibilities:**
- `api/` — published events only. No services, repositories, or model classes.
- `controller/` — request mapping and input validation. No business logic.
- `service/` — all business logic. The only place `ApplicationEventPublisher` is called.
- `repository/` — data access only. No business logic, no event publishing.
- `model/` — domain/entity classes and DTOs, strictly internal to the module. Data crossing
  a module boundary is always carried by an event class in `api/`, never by a model class.
- `OrderEventListener.kt` — a single class per module for all incoming cross-module events.

## Pros and Cons of the Options

### Option A – Custom pragmatic structure

- ✅ Low ceremony — no ports, adapters, or use-case classes required
- ✅ Familiar to Spring Boot developers — minimal learning curve
- ✅ Module's public surface (`api/`), entry points (`controller/`), and external dependencies
  (`OrderEventListener`) are immediately visible
- ✅ Consistent structure across all modules — adding a new module has no ambiguity
- ❌ `model/` can become a catch-all if the convention (domain classes vs. DTOs) is not enforced
- ❌ No strict enforcement of layer responsibilities within a module — relies on code review

### Option B – Hexagonal Architecture (Ports & Adapters)

- ✅ Strict separation between domain logic and infrastructure concerns
- ✅ Domain layer has no framework dependencies — highly testable in isolation
- ❌ Significant structural overhead — ports, adapters, and use-case classes for every feature
- ❌ Ceremony outweighs benefit for our current scope and team size

### Option C – Classic 3-Layer Architecture (global packages)

- ✅ Familiar to most developers
- ❌ Organises by layer globally, not by domain — module cohesion is invisible
- ❌ Cross-cutting changes touch files spread across many top-level packages
- ❌ Module boundaries are not visible in the package structure at all

## Consequences

**Positive:**
- A new developer opening any module can immediately understand its structure
- The single `EventListener` class per module makes inter-module dependencies auditable
- `@ApplicationModuleTest` enforces that nothing outside `api/` is accessed by other modules

**Negative / Trade-offs:**
- `model/` requires active discipline: domain objects and DTOs may coexist there, but both
  must remain strictly internal and must not be referenced by other modules
- Layer responsibilities within a module (e.g. no business logic in controllers) are enforced
  by code review, not by tooling

**Follow-up actions:**
- Document the distinction between domain objects and DTOs within `model/` in the team wiki
- Establish a code review checklist item: no cross-module references outside `api/`
- Use `@ApplicationModuleTest` per module to verify boundary compliance

## Links

- [ADR Index](adr-index.md)
- [ADR-002: Spring Modulith as Application Framework](ADR-002-sprintstart-backend-architecture.md)
- [ADR-003: Event-Based Inter-Module Communication](ADR-003-sprintstart-backend-event-based-architecture.md)
- [Spring Modulith — Module API](https://docs.spring.io/spring-modulith/reference/fundamentals.html#modules.api)
