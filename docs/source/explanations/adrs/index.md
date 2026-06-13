# Architecture Decision Records (ADRs)

An Architecture Decision Record (ADR) documents a significant architectural choice, the context
that led to it, and the consequences of the decision.

## How to Create a New ADR

1. Copy `ADR-NNN-template.md` to `ADR-NNN-<kebab-case-title>.md`
2. Fill in all sections
3. Add a row to the index table below and the toctree
4. Open a PR

## ADR Index

| ADR | Title | Status |
|---|---|---|
| [ADR-NNN-template](ADR-NNN-template.md) | Template | ✅ Accepted |
| [ADR-001-ai-python-toolchain](ADR-001-ai-python-toolchain.md) | Python Toolchain (Version, Package Manager & Project Structure) | ✅ Accepted |
| [ADR-002-backend-architecture](ADR-002-backend-architecture.md) | Backend Architecture | ✅ Accepted |
| [ADR-003-ai-formatter-and-linter](ADR-003-formatter-and-linter.md) | Formatter and Linter | ✅ Accepted |
| [ADR-004-ai-type-checker](ADR-004-ai-type-checker.md) | Python Type Checker | ✅ Accepted |
| [ADR-005-ai-unit-testing](ADR-005-ai-unit-testing.md) | pytest as Testing Framework for the AI Team | ✅ Accepted |
| [ADR-006-sprintstart-backend-architecture](ADR-006-sprintstart-backend-architecture.md) | Spring Modulith as Application Framework | ✅ Accepted |
| [ADR-007-sprintstart-backend-event-based-communication](ADR-007-sprintstart-backend-event-based-communication.md) | Event-Based Inter-Module Communication | ✅ Accepted |
| [ADR-008-sprintstart-backend-module-architecture](ADR-008-sprintstart-backend-module-architecture.md) | Backend Module Structure | ✅ Accepted |
| [ADR-009-sprintstart-backend-language-choice](ADR-009-sprintstart-backend-language-choice.md) | Kotlin as Primary Language | ✅ Accepted |
| [ADR-010-Secrets-Management-Strategy](ADR-010-Secrets-Management-Strategy.md) | Secrets Management Strategy | ✅ Accepted |
| [ADR-011-frontend-framework](ADR-011-frontend-framework.md) | Frontend Framework | ✅ Accepted |
| [ADR-012-ci-cd-tooling](ADR-012-ci-cd-tooling.md) | CI/CD Pipeline | ✅ Accepted |
| [ADR-013-wiki-structure](ADR-013-wiki-structure.md) | Wiki structure and documentation framework | Proposed |

```{toctree}
:hidden:

ADR-NNN-template
ADR-001-ai-python-toolchain
ADR-002-backend-architecture
ADR-003-formatter-and-linter
ADR-004-ai-type-checker
ADR-005-ai-unit-testing
ADR-006-sprintstart-backend-architecture
ADR-007-sprintstart-backend-event-based-communication
ADR-008-sprintstart-backend-module-architecture
ADR-009-sprintstart-backend-language-choice
ADR-010-Secrets-Management-Strategy
ADR-011-frontend-framework
ADR-012-ci-cd-tooling
ADR-013-wiki-structure
```
