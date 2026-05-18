# Working Agreements

This document is the team's shared agreement on **how we work together** —
our Git workflow and how the [Definition of Done](./definition-of-done.md)
is enforced.

---

## 1. Git workflow

### 1.1 Branches

| Branch | Purpose | Protected? |
|---|---|---|
| `main` | Latest working release | Yes — no direct pushes |
| `dev` | Current sprint integration branch | Yes — no direct pushes |
| `feature/<issue>-<short-name>` | One feature or fix; created per issue | No |

Flow:

```
feature/* ──► (PR) ──► dev ──► (PR, merge commit) ──► main
```

- Branch **from `dev`** for every new piece of work.
- Branch names: `feature/<issue-number>-<short-name>`
  *e.g.* `feature/42-onboarding-path-stub`.
- One sprint = one `dev → main` merge, executed as a **merge commit** so each
  sprint boundary is explicit in `main`'s history while the dev commits stay
  preserved.

### 1.2 Pull Requests

| Rule | Detail |
|---|---|
| **Reviewers** | At least 1 approving review from a non-author |
| **CI** | All required checks green (see [Definition of Done](./definition-of-done.md)) |
| **Linked issue** | PR description includes `Closes #<issue>` or `Refs #<issue>` |
| **Description** | Fill in the [DoD checklist](./definition-of-done.md) directly in the PR body |
| **Self-merge** | Allowed after approval + green CI |
| **Force pushes** | Disabled on `dev` / `main`; allowed on feature branches |

### 1.3 Commit messages

- Imperative mood: `add onboarding path generator`, not `added`.
- Reference the issue when relevant: `add onboarding path generator (#42)`.

### 1.4 Conflicts and rebases

- Prefer **rebase onto `dev`** to keep feature branches linear.
- If a conflict can't be resolved trivially, ask the conflicting change's
  author in the team channel before force-resolving.

### 1.5 Enforcement

- **GitHub branch protection** on `dev` and `main`: 1 review required, status
  checks required, no force-push, no direct push.
- **CI** runs on every PR.

---


## 2. Quality enforcement

How the [Definition of Done](./definition-of-done.md) is verified —
automated where possible, manual where it must be judgmental.

| DoD requirement | Mechanism |
|---|---|
| Lint / tests / build / secret-scan | GitHub Actions required CI |
| One review approval | GitHub branch protection on `dev` and `main` |
| All boxes filled on the PR | PR template (Sprint 1+) + reviewer responsibility |
| Black-box test per functional requirement | Reviewer verifies during PR review |


---

