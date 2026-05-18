# ADR-004: [Python Type Checker]

| Field | Value |
|---|---|
| **Status** | `Proposed`  |
| **Date** | 2026-05-18 |
| **Deciders** | AI Team |
| **Supersedes** | None |
| **Superseded by** | – |

---

## Context

The `sprintstart-ai` service uses a modern Python tooling stack with:

- `uv` as package manager
- `ruff` as linter and formatter
- Python 3.12

To improve code quality, maintainability, and developer productivity, the team needs to introduce a static type checker.

The chosen solution should:

- integrate well with modern Python tooling
- support Python 3.12+
- provide fast feedback in local development and CI
- improve reliability for complex AI/RAG logic
- offer strong IDE support
- remain maintainable long-term

This decision affects local development, CI pipelines, onboarding, and overall code quality standards.

## Decision Drivers

- Fast feedback cycles in CI and local development
- Strong type safety for AI and multi-agent logic
- Compatibility with `uv` and `ruff`
- Excellent IDE integration
- Modern Python feature support
- Long-term maintainability and ecosystem adoption
- Low configuration and maintenance overhead

## Considered Options

- **Option A** – `mypy`
- **Option B** – `Pyright`
- **Option C** – `ty` (Astral)
- **Option D** – `Pyrefly`
## Decision

**Chosen option: Option B – `Pyright`**

Pyright provides the best balance of performance, maturity, strict type checking, and developer experience for a modern Python project.


## Rationale

Pyright is currently one of the fastest and most actively maintained Python type checkers.

It integrates well with our chosen tooling stack:

- complements `ruff` by focusing solely on type analysis
- works well alongside `uv`
- offers excellent IDE support, especially through VS Code Pylance
- supports modern Python typing features quickly
- performs significantly faster than `mypy` in larger projects

Although newer tools such as `ty` are promising, they are still too experimental for a long-term university project with multiple contributors.


## Pros and Cons of the Options

### Option A – `mypy`

- ✅ Very mature and widely adopted
- ✅ Large ecosystem and extensive documentation
- ✅ Proven in large production codebases
- ❌ Slower on larger projects
- ❌ Configuration can become complex
- ❌ Slower adoption of newer Python typing features


### Option B – `Pyright` (Chosen)

- ✅ Extremely fast
- ✅ Excellent Python 3.12+ support
- ✅ Great IDE integration
- ✅ Strong strict-mode support
- ✅ Simple and modern developer experience
- ✅ Well suited for CI pipelines
- ❌ Configuration is less Python-native
- ❌ Relies partly on Microsoft tooling ecosystem


### Option C – `ty` (Astral)

- ✅ Very modern approach
- ✅ Fits well into the Astral ecosystem (`uv`, `ruff`)
- ✅ Potentially very high performance
- ❌ Still experimental
- ❌ Small ecosystem and limited adoption
- ❌ Higher risk of breaking changes


### Option D – `Pyrefly`

- ✅ Interesting alternative approach
- ❌ Very small community
- ❌ Limited production usage
- ❌ Unclear long-term support
- ❌ Higher migration risk in the future


## Consequences

**Positive:**
- Faster feedback during development
- Earlier detection of bugs and integration issues
- Better maintainability for AI and RAG components
- Improved onboarding through clearer interfaces and typing
- High-performance type checking in CI

**Negative / Trade-offs:**
- Additional tool in the development toolchain
- Team members need basic familiarity with static typing
- Separate configuration file may be required

**Follow-up actions:**
- [ ] Add `pyright` configuration
- [ ] Integrate Pyright into CI pipeline
- [ ] Define strictness level (`strict` recommended)
- [ ] Add IDE setup documentation
- [ ] Optionally integrate into pre-commit hooks

## Links

- [ADR Index](adr-index.md)
- https://microsoft.github.io/pyright/
