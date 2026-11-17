# Repo Adapter Template (`.agents/rules/kanban-workflow.md`)

Copy this template into `<repo>/.agents/rules/kanban-workflow.md` and replace every `<PLACEHOLDER>`. Because it uses `trigger: model_decision`, the host only loads these IDs and repo commands when working with the Kanban board or delivery workflow.

```markdown
---
trigger: model_decision
description: GitHub Project #<PROJECT_NUMBER> Kanban workflow (Backlog -> Review -> In Progress -> In Review -> Done) for <OWNER>/<REPO> tickets, PO refinement, role-scoped Dev execution, conditional adversarial review, and PR merge rules.
---

# <PROJECT_NAME> GitHub Kanban Workflow Rules

- **Repository:** `<OWNER>/<REPO>`
- **GitHub Project:** `#<PROJECT_NUMBER>` — *<PROJECT_TITLE>* (`<PROJECT_ID>`, https://github.com/users/<OWNER>/projects/<PROJECT_NUMBER>)
- **Status Field ID:** `<STATUS_FIELD_ID>`
  - `Backlog`: `<BACKLOG_OPTION_ID>`
  - `Review`: `<REVIEW_OPTION_ID>`
  - `In Progress`: `<IN_PROGRESS_OPTION_ID>`
  - `In Review`: `<IN_REVIEW_OPTION_ID>`
  - `Done`: `<DONE_OPTION_ID>`

> Retrieve exact IDs with `gh project list --owner <OWNER>` and `gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json`.

---

## Repo Verification Commands

Run the applicable commands inside every `In Progress` PR before requesting `In Review`:

```bash
# Example (replace with your project's commands)
# npm ci && npx tsc --noEmit && npm run build && npm test
```

- **Preview / Test Artifact:** `<e.g., Netlify Deploy Preview URL on PR / APK debug build + emulator screenshot>`

---

## High-Risk Paths (Triggers Conditional Adversarial Review)

Automatically run a read-only `sec-privacy` / `qa-tester` adversarial review pass before moving a PR to `In Review` if the ticket is tagged `Risk Tier: high-risk` **or** `gh pr diff --name-only` touches any of:
- `<e.g., supabase/migrations/**>`
- `<e.g., **/auth/**, **/security/**, **/rls/**>`
- `<e.g., server/src/game/visibility*, **/billing/**>`

---

## Domain Specialist Roles (`flash` Subagents)

Map issue `Target Role` tags to the repo's specialists:
- `frontend` / `designer`: `<UI stack & design skill path, e.g., .agents/skills/project-ui/SKILL.md>`
- `backend-dev`: `<Server/DB stack, e.g., Node/Fastify/Postgres or Supabase RPCs>`
- `qa-tester`: `<Test frameworks, e.g., Vitest, Playwright, Cypress>`
- `sec-privacy`: `<Security invariants, e.g., RLS isolation, zero-PII logs, visibility matrix>`

---

## Stage Rules (Summary)

1. **`Backlog`:** Intake via `.github/ISSUE_TEMPLATE/`. Items marked `ready` wait here. Never implement `Backlog` items automatically.
2. **`Review` (Flash PO Pass):** Check `docs/DECISIONS.md` (if present); split multi-module prompts into atomic issues; if missing decisions, label `need info` + post numbered questions; if clear, rewrite body (`User Story`, `Acceptance Criteria`, `Likely Files / Area`, `Agent Metadata`), add `ready`, remove `need info`, and move back to `Backlog`.
3. **`In Progress`:** Create branch (`feat/<num>-<slug>` / `fix/<num>-<slug>`), delegate to the matching `flash` subagent (1–2 line trivial fixes may be done directly), verify tests, open PR with `Closes #<num>`, run Lead `gh pr diff` audit loop (+ conditional adversarial review on high-risk paths), and move card to `In Review`.
4. **`In Review`:** Human gate. **Never** merge or move to `Done` until the PR is approved on GitHub or the user writes **`approve and merge`**.
5. **`Done`:** Run `gh pr merge <pr> --squash --delete-branch`, move card to `Done`, update `docs/PROGRESS.md` (and `docs/ROADMAP.md` / `CHANGELOG.md` / SemVer tag on milestones).
```
