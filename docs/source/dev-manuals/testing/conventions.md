# Testing Conventions & Standards

## Testing Levels

The following levels are defined for SprintStart. Unit and integration tests
are mandatory from the start. E2E and contract tests will be introduced once
the stack is more stable.

| Level | Scope | Status |
|---|---|---|
| Unit Tests | Single functions / components | Active |
| Integration Tests | Interaction between modules | Active |
| E2E Tests | Full frontend user flows | Planned |
| Contract Tests | Frontend ↔ Backend, Backend ↔ AI | Planned |

Which levels are enforced per repo is decided by each team.

## Minimum Test Requirements

- Every requirement (issue) must have **at least one black-box test**,
  tested against the acceptance criteria of that issue.
- Critical failures must not occur in production.

## Code Quality

- **Linter & Formatter** are required for all repos.
- For languages without built-in type checking, a **static type checker**
  must be added (e.g. mypy for Python, tsc for JS/TS).

## Code Coverage

No hard PR rejection threshold.
Teams are encouraged to use this as a discussion tool, not a gate.
