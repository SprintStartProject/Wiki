# ADR-003: Formatter and Linter

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-18 |
| **Deciders** | AI Team |
| **Supersedes** | None |
| **Superseded by** | – |

---

## Context

The `sprintstart-ai` service is built in Python 3.12 and uses `uv`
with `pyproject.toml` for dependency management.

Before development continues, the team needs to agree on a formatter
and linter setup to ensure consistent code quality across local
development and CI.

This decision affects:

1. **Code formatting** — ensures consistent style and readability across the project
2. **Linting** — catches common issues, unused imports, and style violations early
3. **CI quality gates** — enforces consistent standards before merging code
4. **Developer onboarding** — simplifies local setup and development workflows

This decision blocks the final CI quality pipeline setup.

## Decision Drivers

- Consistent code style across the project
- Fast feedback in local development and CI
- Simple configuration in `pyproject.toml`
- Low maintenance overhead
- Compatibility with Python 3.12
- Good integration with `uv` and GitHub Actions

## Considered Options

### Formatter / Linter

- **Option A** – Ruff
- **Option B** – Black + isort + Flake8
- **Option C** – Pylint + Black + isort

## Decision

**Formatter and linter: Ruff**

Ruff will be configured in `pyproject.toml`
and executed in CI using:

```bash
uv run ruff check .
uv run ruff format --check .
```

Selected Ruff configuration:

```toml
[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I"]
```

## Rationale

Ruff is the best option for our project because it combines
linting, formatting, and import sorting into a single fast tool.

It integrates well with our existing `uv` + `pyproject.toml`
setup and reduces tooling complexity compared to maintaining
multiple separate tools.

Its performance also improves developer experience and
CI execution speed.

## Pros and Cons of the Options

### Formatter / Linter

#### Option A – Ruff

- ✅ Extremely fast
- ✅ Combines linting, formatting, and import sorting
- ✅ Simple configuration in `pyproject.toml`
- ✅ Works well with `uv` and GitHub Actions
- ✅ Reduces the number of required tools
- ❌ Relatively new compared to Black or Flake8
- ❌ Some team members may be unfamiliar

#### Option B – Black + isort + Flake8

- ✅ Mature and widely used
- ✅ Clear separation of responsibilities
- ✅ Familiar to many Python developers
- ❌ Requires multiple tools and configurations
- ❌ More maintenance overhead
- ❌ Slower CI execution compared to Ruff

#### Option C – Pylint + Black + isort

- ✅ Very detailed static analysis
- ✅ Mature and feature-rich
- ✅ Highly configurable
- ❌ More complex setup and configuration
- ❌ Can be overly strict for early-stage projects
- ❌ Slower feedback during development and CI

## Consequences

**Positive:**

- Consistent formatting across the entire codebase
- Faster CI quality checks
- Simpler developer setup
- Import ordering handled automatically
- Reduced tooling complexity

**Negative / Trade-offs:**

- Team members unfamiliar with Ruff may require a short ramp-up
- Ruff behavior may differ slightly from Black or Flake8 defaults

**Follow-up actions:**

- [x] Add Ruff to dev dependencies
- [x] Configure Ruff in `pyproject.toml`
- [x] Add Ruff lint check to GitHub Actions
- [x] Add Ruff format check to GitHub Actions
- [ ] Document local Ruff usage in README

## Links

- [ADR Index](index.md)
- [sprintstart-ai](https://github.com/SprintStartProject/sprintstart-ai)
