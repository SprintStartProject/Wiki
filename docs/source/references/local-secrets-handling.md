# Local secrets handling

## Purpose

This page describes how local secrets are handled during development.

For the overall strategy and rationale, see [ADR-010 (Secrets Management Strategy)](../explanations/adrs/adr-010-secrets-management-strategy.md).

## Local development

Each repository contains a `.env.example` file.

To create your local configuration:

1. Copy `.env.example` to `.env`
2. Replace placeholder values with your local values
3. Keep the `.env` file on your machine only

Example:

```bash
cp .env.example .env
```

The `.env` file is ignored by Git and must never be committed to the repository.

## Scope — Local development only

`.env` files are only intended for local development.

They must not be used for:

* CI pipelines
* Production deployments
* Shared team environments

## Related stories

### CI secrets

Secrets used during CI execution are managed separately through GitHub Actions secrets.

See: [#126 – CI Secrets](https://github.com/SprintStartProject/Wiki/issues/126)

### Runtime / Deployment secrets

Secrets used in deployed environments are managed separately through the runtime secret store.

See: [#127 – Runtime Secrets](https://github.com/SprintStartProject/Wiki/issues/127)
