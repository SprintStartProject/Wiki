# ADR-001: Python Toolchain (Version, Package Manager & Project Structure)

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-05-11 |
| **Deciders** | AI Team |
| **Supersedes** | None |
| **Superseded by** | – |

---

## Context

The `sprintstart-ai` service is built in Python. Before any code is written, the team needs to
agree on:

1. **Python version** — affects available language features, library compatibility, and Docker base images
2. **Package manager / project structure tool** — determines how dependencies are declared, locked, and installed in CI and production

This decision blocks the CI pipeline setup and the local dev setup.

## Decision Drivers

- Reproducible, deterministic builds in CI and across developer machines
- Modern Python tooling with good ecosystem support
- Low friction for team members onboarding to the project
- Lock file support

## Considered Options

### Python Version
- **Option A** – Python 3.11
- **Option B** – Python 3.12
- **Option C** – Python 3.13

### Package Manager
- **Option A** – `uv` + `pyproject.toml`
- **Option B** – `Poetry` + `pyproject.toml`
- **Option C** – `pip` + `requirements.txt` + `requirements-dev.txt`

## Decision

**Python version: 3.12**
**Package manager: uv + pyproject.toml**

## Rationale

3.12 is the best Option for our Project as it combines performance with stability.
We choose to combine it with uv as the package manager, as it brings everything we need and is fast.

## Pros and Cons of the Options

### Python Version

#### Option A – Python 3.11

- ✅ Stable, widely adopted, excellent library support
- ✅ Available on all major Docker base images
- ✅ Most team members likely familiar
- ❌ Missing some newer features (e.g. improved `typing` in 3.12+)

#### Option B – Python 3.12

- ✅ Significant performance improvements over 3.11
- ✅ Improved error messages, better f-string support
- ✅ Still within active support window
- ❌ Some libraries may lag slightly behind on 3.12 support

#### Option C – Python 3.13

- ✅ Latest stable release, newest language features
- ❌ Newest = least tested by the ecosystem
- ❌ Some dependencies may not yet support 3.13
- ❌ Higher risk for a project starting now

### Package Manager

#### Option A – `uv` + `pyproject.toml`

- ✅ Extremely fast (written in Rust)
- ✅ Drop-in replacement for pip + venv
- ✅ Lock file support
- ✅ Single tool for venv, install, and lock
- ✅ Growing rapidly in adoption
- ❌ Relatively new
- ❌ Some team members may be unfamiliar

#### Option B – Poetry + `pyproject.toml`

- ✅ Mature, widely used in production Python projects
- ✅ Lock file support
- ✅ Good CI integration and documentation
- ✅ Handles both dependency management and packaging
- ❌ Slower than uv
- ❌ Requires separate installation, slightly heavier setup

#### Option C – `pip` + `requirements.txt`

- ✅ No learning curve as it is known by most
- ✅ Maximum compatibility
- ❌ No lock file by default
- ❌ Dev vs production dependencies harder to separate cleanly
- ❌ Not considered modern practice for new projects in 2026

## Consequences

**Positive:**
- Unblocks CI pipeline setup and local dev setup documentation
- Enables deterministic builds across all developer machines and CI

**Negative / Trade-offs:**
- Team members unfamiliar with the chosen tool will need brief ramp-up

**Follow-up actions:**

- [ ] Update docs with installation instructions
- [ ] Configure CI pipeline with chosen toolchain
- [ ] Create `pyproject.toml` or `requirements.txt` in `sprintstart-ai` repo
- [x] Update ADR status from `Proposed` to `Accepted`
- [ ] Choose and configure linter/formatter

## Links

- [ADR Index](adr-index.md)
- [sprintstart-ai](https://github.com/SprintStartProject/sprintstart-ai)
