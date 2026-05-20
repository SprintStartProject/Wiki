# ADR-005: pytest as Testing Framework for the AI Team

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-19 |
| **Deciders** | AI Team |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

The AI Team develops Python-based components (model pipelines, data processing, API integrations) and requires a unified testing framework. A framework is needed that is easy to adopt, integrates well with the Python ecosystem, and covers both unit and integration testing.

## Decision Drivers

- Simple syntax and low barrier to entry for the entire team
- Good integration with existing tooling
- Compatibility with AI-specific libraries
- Active community and long-term support
- Support for fixtures, parametrization, and a rich plugin ecosystem

## Considered Options

- **Option A** – pytest
- **Option B** – unittest (Standard Library)
- **Option C** – nose2

## Decision

**Chosen option: Option A – pytest** – pytest offers the best balance of developer ergonomics, ecosystem integration, and flexibility for the specific needs of an AI team.

## Rationale

pytest stands out from the alternatives through its clean syntax (no mandatory class structure, plain `assert` statements) and its powerful fixture system. In an AI/ML context, easy test parametrization — e.g. for different model configurations or datasets — is a decisive advantage. The plugin ecosystem (`pytest-cov`, `pytest-mock`, `pytest-asyncio`, etc.) covers all common requirements without introducing additional frameworks. Unlike `unittest`, pytest encourages readable, idiomatic Python test code. `nose2` offers no meaningful advantage over pytest and has a significantly smaller, less active community.

## Pros and Cons of the Options

### Option A – pytest

- ✅ Readable syntax with plain `assert` statements and no boilerplate
- ✅ Powerful fixture system with scope control (function, class, module, session)
- ✅ Rich plugin ecosystem (`pytest-cov`, `pytest-mock`, `pytest-asyncio`, `pytest-xdist`)
- ✅ First-class support for parametrized tests — ideal for ML experiments
- ✅ Full backward compatibility: existing `unittest` tests run without modification
- ❌ External package — not part of the standard library, requires installation
- ❌ Advanced features (e.g. nested fixtures) have a learning curve

### Option B – unittest (Standard Library)

- ✅ Part of the Python standard library, no installation required
- ✅ Familiar xUnit-style structure for developers from Java/C# backgrounds
- ❌ Verbose syntax requiring class inheritance and `self.assertEqual`-style assertions
- ❌ No native plugin ecosystem, limited extensibility
- ❌ Parametrization is cumbersome (e.g. via `subTest` or third-party libraries)

### Option C – nose2

- ✅ Extends `unittest` with improved plugin support
- ✅ Automatic test discovery
- ❌ Significantly smaller community and reduced development activity compared to pytest
- ❌ No meaningful advantage over pytest at a higher onboarding cost
- ❌ Weaker integration with modern CI/CD tooling

## Consequences

**Positive:**
- A unified testing standard across the AI Team improves maintainability and code quality
- Straightforward CI/CD integration via the `pytest` CLI
- Code coverage reporting directly through `pytest-cov` with minimal configuration
- Async tests (e.g. for API integrations) are supported out of the box via `pytest-asyncio`
- New team members can get up to speed quickly thanks to clear syntax and excellent documentation

**Negative / Trade-offs:**
- pytest must be added as a dependency to all projects
- Fixture scope and ordering can introduce subtle bugs in complex test setups
- The team needs to define conventions for fixture usage to avoid inconsistency

**Follow-up actions:**
- Add `pytest` to `pyproject.toml`
- Document team-internal testing guidelines and naming conventions

## Links

- [ADR Index](index.md)
- [pytest Documentation](https://docs.pytest.org/)
- [pytest-asyncio](https://pytest-asyncio.readthedocs.io/)
