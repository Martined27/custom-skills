# GitHub Issue & Pull Request Templates

Drop-in templates for intake (`.github/ISSUE_TEMPLATE/`), Flash Product Owner refinement in `Review`, `need info` clarification loops, and PR submissions.

---

## 1. `.github/ISSUE_TEMPLATE/bug.yml`

```yaml
name: Bug Report
description: Report an unexpected error or broken behavior
title: "[Bug]: "
labels: ["bug"]
body:
  - type: input
    id: area
    attributes:
      label: Affected screen, route, or module
      description: e.g., /admin/review, auth middleware, mobile lobby screen
      placeholder: /admin/review
    validations:
      required: true
  - type: textarea
    id: description
    attributes:
      label: Actual vs. Expected behavior
      description: Briefly describe what happens today and what should happen instead.
      placeholder: "Clicking Save throws 500, but it should persist the change and show a confirmation toast."
    validations:
      required: true
  - type: textarea
    id: steps
    attributes:
      label: Steps to reproduce / context (optional)
      description: Screenshots, test role/user, payloads, or reproduction steps.
      placeholder: "1. Sign in as...\n2. Navigate to..."
    validations:
      required: false
```

---

## 2. `.github/ISSUE_TEMPLATE/enhancement.yml`

```yaml
name: Enhancement / Feature
description: Propose a new feature or improvement
title: "[Enhancement]: "
labels: ["enhancement"]
body:
  - type: input
    id: area
    attributes:
      label: Screen, route, or module
      description: e.g., /profile, API webhooks, DB/RLS, CI pipeline
      placeholder: /profile
    validations:
      required: true
  - type: textarea
    id: goal
    attributes:
      label: Goal (raw prompt or idea)
      description: Explain what you want to achieve; the Product Owner pass in Review will refine scope and acceptance criteria.
      placeholder: "I want users to be able to..."
    validations:
      required: true
  - type: textarea
    id: notes
    attributes:
      label: Constraints or important rules (optional)
      description: Role permissions, localization requirements, edge cases, or UI expectations.
    validations:
      required: false
```

---

## 3. PO-Refined Issue Body Template (`Review` → `ready`)

Use this structure when expanding a clear ticket in `Review` before moving it back to `Backlog` with label `ready`:

```markdown
## User Story
As a <role/actor>, I want <capability>, so that <measurable outcome>.

## Context & Constraints
- <Original prompt details, edge cases, or UI/i18n rules>
- **ADR Alignment:** <None | Follows ADR-XXX; does not alter locked stack>

## Acceptance Criteria
- [ ] <Observable, testable requirement 1>
- [ ] <Observable, testable requirement 2>
- [ ] <Error/edge case handling + automated test coverage>

## Likely Files / Area
- `<path/to/file1>` — <why this file is relevant>
- `<path/to/file2>` — <why this file is relevant>

## Agent Metadata
- **Target Role:** `<backend-dev | frontend | designer | mobile-dev | qa-tester>`
- **Risk Tier:** `<standard | high-risk>` *(high-risk if touching auth, RLS, SQL migrations, visibility/privacy, or billing/points math)*
```

---

## 4. `need info` Comment Template (`Review`)

Post this comment when product decisions or missing context block safe scoping:

```markdown
### Needs Clarification (`need info`)

Before moving this ticket to `ready`, please clarify:

1. **<Topic 1>:** <Concise question with concrete options if helpful (e.g., Option A vs. Option B)>
2. **<Topic 2>:** <Concise question>

_Leaving this card in `Review` until answered._
```

---

## 5. Pull Request Body Template (`In Progress` → `In Review`)

```markdown
Closes #<ISSUE_NUMBER>

## Summary
- <Concise change summary 1>
- <Concise change summary 2>

## Verification Run
- [ ] Typecheck / Lint (`<exact command>` — `<result>`)
- [ ] Unit / Integration Tests (`<exact command>` — `<X passed, 0 failed>`)
- [ ] Build / Preview (`<Deploy Preview URL or build artifact status>`)

## Audits
- **Lead Diff Audit (`gh pr diff`):** <Verified against Acceptance Criteria and DECISIONS.md>
- **Adversarial Review:** <Not required (`Risk Tier: standard`) | Completed by `sec-privacy`/`qa-tester` — 0 unresolved findings>

## How to Test (Human Gate)
1. <Step 1 using Deploy Preview / local app>
2. <Expected result>

> Reply **`approve and merge`** (or approve on GitHub) when ready to squash-merge into `main` and move `#<ISSUE_NUMBER>` to `Done`.
```
