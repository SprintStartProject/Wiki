# CI Pipeline Setup Guide

## Purpose

This document describes how to create and maintain Continuous Integration (CI) pipelines for new repositories within the SprintStart project.

The goal of the CI pipeline is to automatically validate code quality, detect security issues, verify successful builds, and prevent broken code from being merged into shared branches.

---

# CI Principles

Every repository should follow the same validation flow:

```text
Source Code
    ↓
Secret Scan
    ↓
Code Quality Checks
    ↓
Automated Tests
    ↓
Build Verification
    ↓
Container Build Validation
```

All validation steps must succeed before a pull request can be merged.

---

# Repository Structure

Create a workflow file inside:

```text
.github/workflows/ci.yml
```

GitHub Actions automatically discovers workflow files stored in this directory. GitHub workflows are defined using YAML and consist of triggers, jobs, and steps.

---

# Workflow Triggers

Every CI pipeline should run on:

```yaml
on:
  push:
  pull_request:
```

This ensures that validation runs both when developers push changes and when pull requests are created or updated.

---

# Stage 1: Secret Scanning

Every repository must include a Gitleaks scan.

Example:

```yaml
gitleaks:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v4

    - name: Download Gitleaks
      run: |
        wget https://github.com/gitleaks/gitleaks/releases/download/v8.30.1/gitleaks_8.30.1_linux_x64.tar.gz
        tar -xzf gitleaks_8.30.1_linux_x64.tar.gz

    - name: Scan repository
      run: ./gitleaks detect --verbose --redact
```

Purpose:

* Detect API keys
* Detect passwords
* Detect access tokens
* Prevent accidental secret exposure

The workflow must fail if a secret is detected.

---

# Stage 2: Code Quality Checks

Each repository must execute language-specific quality checks.

## Kotlin / Spring Boot

```yaml
- run: ./gradlew ktlintCheck
- run: ./gradlew detekt
```

## Node.js

```yaml
- run: npm run lint
```

## Python

```yaml
- run: uv run ruff check .
- run: uv run ruff format --check .
- run: uv run pyright src/
```

Purpose:

* Enforce coding standards
* Detect potential bugs
* Improve maintainability

---

# Stage 3: Automated Testing

Automated tests must run before build verification.

## Backend

```yaml
- run: ./gradlew test
```

## Python

```yaml
- run: uv run pytest
```

Tests should fail the workflow if any assertion fails.

---

# Stage 4: Build Verification

The repository must successfully build.

## Backend

```yaml
- run: ./gradlew assemble
```

## Frontend

```yaml
- run: npm run build
```

Purpose:

* Verify compilation
* Detect dependency issues
* Ensure deployable artifacts can be generated

---

# Stage 5: Docker Build Validation

Repositories using Docker must validate image creation.

Example:

```yaml
- run: docker build -t project-name .
```

Purpose:

* Verify Dockerfile correctness
* Detect packaging issues
* Ensure deployment readiness

---

# Branch Protection

The project uses a controlled merge strategy.

Allowed merge flow:

```text
feature/* → dev → main
                ↑
          hotfix/*
```

Direct pull requests from feature branches to `main` are not permitted.

Example validation workflow:

```yaml
on:
  pull_request:
    branches:
      - main

jobs:
  check-branch:
    runs-on: ubuntu-latest

    steps:
      - name: Check source branch
        run: |
          if [[ "${GITHUB_HEAD_REF}" != "dev" ]] && [[ "${GITHUB_HEAD_REF}" != hotfix/* ]]; then
            exit 1
          fi
```

This ensures that only approved integration branches can be merged into production.

---

# Creating a New Repository CI Pipeline

When creating a new repository:

1. Create `.github/workflows/ci.yml`
2. Add workflow triggers (`push`, `pull_request`)
3. Add Gitleaks scanning
4. Add language-specific linting
5. Add automated tests
6. Add build verification
7. Add Docker validation (if applicable)
8. Verify successful execution in GitHub Actions

---

# Failure Handling

Common CI failures:

| Failure Type    | Cause                        |
| --------------- | ---------------------------- |
| Gitleaks        | Secret detected              |
| Lint            | Style violation              |
| Static Analysis | Code quality issue           |
| Tests           | Failing assertions           |
| Build           | Compilation error            |
| Docker          | Invalid Docker configuration |

Developers must resolve all failures before merging changes.
