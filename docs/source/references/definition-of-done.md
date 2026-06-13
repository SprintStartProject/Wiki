# Definition of Done

Copy the section below into every Pull Request description. Tick what applies or leave unchecked if it doesnt apply.

---


**Type of PR** — pick one:

- [ ] Functional       — adds or changes user-visible behavior
- [ ] Non-functional   — improves a measurable property (perf, security, a11y …)
- [ ] Mixed            — both
- [ ] Internal         — refactor / docs / tests only

---

## 1 Process baseline — every box must be ticked, on every PR

- [ ] All acceptance criteria in the linked issue are met
- [ ] 1 review approval from a non-author
- [ ] CI green: lint, type-check, unit tests, build, secret-scan
- [ ] No secrets / tokens / credentials in the diff
- [ ] No new `TODO` / `FIXME` without a follow-up issue
- [ ] Docs updated if behavior, API, config, or architecture changed
- [ ] **No regression** of other NFR baselines (a11y / perf / security)

---

## 2 Outcome proof — fill the bullets that match your PR type

**Functional / Mixed PR:**

- [ ] ≥ 1 **black-box test** that exercises the acceptance criteria
  *e.g. upload a markdown file, then assert the chat answer cites it*


**Non-functional / Mixed PR:**

- [ ] **Measurement** that proves the acceptance criteria, attached to the PR
  *e.g. benchmark output for `chat p95 < 2 s`, axe audit for `a11y ≥ 90`, scan report for `0 critical CVEs`*


---

## 3 Cross-cutting impact — tick the areas this PR touches

For each area below, ask: **"does my PR touch this?"**

-> If **yes** → tick the area and complete its sub-checks (they become mandatory).\
-> If **no** → skip it.\
-> If nothing applies → tick *"None of the above"*.


- [ ] **UI touched**
  - [ ] Responsive on desktop **and** mobile
  - [ ] Contrast + visible focus state, no color-only signals
  *e.g. new button has a focus ring and is readable on a 360 px screen*

- [ ] **Backend endpoint added or changed**
  - [ ] Behind authentication — not accidentally public
  - [ ] Input validation (size / type / allowed values)
  - [ ] OpenAPI spec updated
  *e.g. `/upload` rejects files > 10 MB with a clear error*

- [ ] **Stores user data, ingested content, or LLM output**
  - [ ] Logged with the standard fields (request id, source, latency)
  - [ ] No PII / credentials in log output


- [ ] **New env var or config**
  - [ ] Added to `.env.example` and mentioned in `README.md`
  - [ ] No real value committed
  *e.g. `LLM_API_KEY=...` — only the key name lives in `.env.example`*

- [ ] **Deployment / build changed**
  - [ ] Deployable via the standard pipeline (no undocumented manual steps)
  - [ ] Local dev setup still works


- [ ] **None of the above** — purely internal change

---
