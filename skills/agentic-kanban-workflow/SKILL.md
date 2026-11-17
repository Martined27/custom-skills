---
name: agentic-kanban-workflow
description: Orchestrate a token-efficient, quota-resilient AI software delivery workflow combining a 5-stage GitHub Projects Kanban (Backlog -> Review -> In Progress -> In Review -> Done), Flash Product Owner triage, role-scoped subagent execution, Lead Architect PR diff auditing, conditional adversarial security/QA reviews, and repo-native session handoff docs (PROGRESS.md, DECISIONS.md, ROADMAP.md).
argument-hint: "Board action or stage to process (e.g., 'review all items in Review', 'implement issue #12', 'audit PR #15', or 'bootstrap workflow in this repo')"
---

# Agentic Kanban Workflow (`agentic-kanban-workflow`)

Coordinate multi-agent software delivery from raw idea intake to verified release while minimizing expensive reasoning tokens and surviving session/quota interruptions cleanly.

See also:
- [references/kanban-workflow-rule.template.md](references/kanban-workflow-rule.template.md) — Per-repo `.agents/rules/kanban-workflow.md` (`trigger: model_decision`) starter template
- [references/issue-and-pr-templates.md](references/issue-and-pr-templates.md) — GitHub Issue forms (`bug.yml`, `enhancement.yml`), PO-refined issue structure, `need info` comment format, and PR template
- [references/project-docs-templates.md](references/project-docs-templates.md) — Repo-native continuity templates (`AGENTS.md`, `docs/PROGRESS.md`, `docs/DECISIONS.md`, `docs/ROADMAP.md`)
- [references/gh-cli-playbook.md](references/gh-cli-playbook.md) — Exact `gh issue`, `gh pr`, and `gh project item-edit` commands

---

## 1. Two-Layer Architecture & Tracker Modes

Keep the universal workflow portable while isolating repository-specific IDs, commands, and risk surfaces into a thin repo adapter.

### Two-Layer Split
1. **Layer 1 — Generic Skill (`SKILL.md`):** Holds the universal 5-stage Kanban lifecycle, agent tiering, PR audit loop, merge gate, and session handoff rules.
2. **Layer 2 — Repo Adapter Rule (`.agents/rules/kanban-workflow.md`):** Uses `trigger: model_decision` frontmatter so it only loads into context when working with the board or delivery workflow. It stores:
   - GitHub Project number, Project ID (`PVT_...`), Status Field ID (`PVTSSF_...`), and column Option IDs
   - Verification commands (typecheck, lint, unit/E2E tests, build)
   - Preview mechanism (e.g., Netlify Deploy Preview, Vercel preview, emulator screenshot, or staging URL)
   - High-risk path globs that trigger conditional adversarial review
   - Domain specialist role names tailored to the repository

### Supported Tracker Modes

| Mode | Source of Truth | Best For |
| :--- | :--- | :--- |
| **GitHub Projects Kanban** | GitHub Issues + Projects v2 (`Backlog` → `Review` → `In Progress` → `In Review` → `Done`) | Active feature/bug queues with human triage and Deploy Previews |
| **Repo-Docs Kanban** | `docs/ROADMAP.md` + `docs/PROGRESS.md` + `docs/DECISIONS.md` | Offline work, early greenfield sprints, or repos without a GitHub Project board |
| **Hybrid (Recommended)** | GitHub Projects v2 for live ticket state + `docs/` triad updated at session boundaries/milestones | Multi-session projects needing both visual PR gating and quota-safe local state |

> **Bootstrapping a repo:** When asked to set up this workflow in a repository, inspect `.agents/rules/`, `.github/ISSUE_TEMPLATE/`, and `docs/`, then scaffold any missing files from [references/kanban-workflow-rule.template.md](references/kanban-workflow-rule.template.md), [references/issue-and-pr-templates.md](references/issue-and-pr-templates.md), and [references/project-docs-templates.md](references/project-docs-templates.md).

---

## 2. Multi-Agent Team Roles & Tiering

Assign work according to reasoning cost and blast radius:

| Role | Model Tier | Core Responsibilities |
| :--- | :--- | :--- |
| **Product Owner (PO)** | Lightweight (`flash`) | Operates on `Review` items. Checks `docs/DECISIONS.md` for ADR compliance, splits oversized prompts into atomic issues, asks numbered questions under `need info`, or writes a self-contained spec (**User Story**, **Acceptance Criteria**, **Likely Files / Area**, **Target Role**, **Risk Tier**) and labels `ready`. |
| **Lead Architect & PR Reviewer** | High-Reasoning (Lead) | Orchestrates the board/roadmap, spawns scoped `flash` workers, audits `gh pr diff` against Acceptance Criteria and ADRs, sends actionable revision feedback back to the worker instead of rewriting bulk code directly, enforces the human merge gate, runs `gh pr merge --squash --delete-branch`, and updates `CHANGELOG.md` / `docs/PROGRESS.md` / SemVer tags. |
| **Domain Execution Subagent** | Lightweight (`flash`) | Specializes by repo domain (e.g., `frontend`/`designer`, `backend-dev`, `mobile-dev`, `qa-tester`). Never commits to `main`. Works on `feat/<issue>-<slug>`, `fix/<issue>-<slug>`, or `chore/<issue>-<slug>`, writes code + tests, runs local verification, commits, pushes, and opens a PR (`Closes #<num>`). |
| **Conditional Adversarial Reviewer** | Lightweight (`flash`, read-only) | Triggered **only** when a ticket is tagged `high-risk` or its diff touches high-risk paths configured in `.agents/rules/kanban-workflow.md` (e.g., RLS, SQL migrations, Auth, privacy/visibility matrices, billing/points math). Audits the diff and reports findings before the card enters `In Review`. |

> **Trivial-Fix Exception:** A targeted 1–2 line fix (copy change, typo, config tweak) with `standard` risk may be implemented directly by the active agent without spawning a separate subagent—while still following the branch, verification, PR, and human merge gate rules.

---

## 3. The 5-Stage Kanban Lifecycle

```text
[Backlog (raw idea / template)]
        │
        ▼ (user moves card to Review)
[Review — Flash Product Owner Pass]
   ├── Ambiguous / missing decisions ──► Label `need info` + numbered comment (stays in Review)
   ├── Multi-module / oversized      ──► Split into atomic sub-issues before marking `ready`
   └── Atomic & clear (+ ADR check)  ──► Expand Story + AC + Likely Files + Role/Risk
                                         ──► Label `ready` ──► Move back to [Backlog]
                                                                      │
                                                                      ▼ (user moves card to In Progress)
                                                          [In Progress — Branch, Dev & Lead Audit]
                                                             1. Subagent implements on branch & runs tests
                                                             2. Opens PR (`Closes #<num>`)
                                                             3. Lead audits `gh pr diff` (loops back to subagent if fixes needed)
                                                             4. If `high-risk`, run Adversarial Reviewer pass
                                                                      │
                                                                      ▼ (checks + audit pass)
                                                          [In Review — PR & Deploy Preview Gate]
                                                             ⚠️ STRICT HUMAN GATE: wait for GitHub approval
                                                                or literal user command `approve and merge`
                                                                      │
                                                                      ▼ (approved)
                                                          [Done — Squash Merge & Continuity Sync]
```

### Stage 1: `Backlog` (Intake & Ready Queue)
- **Entry:** User files raw bugs, feature prompts, or improvements using `.github/ISSUE_TEMPLATE/bug.yml` or `enhancement.yml`. Refined tickets also return here tagged `ready`.
- **Rule:** Never implement `Backlog` items automatically. Wait until the user moves an item to `Review` or `In Progress` (or explicitly selects it).

### Stage 2: `Review` (Flash Product Owner Pass)
When asked to process `Review` (batch all items in `Review` together using [references/gh-cli-playbook.md](references/gh-cli-playbook.md)):
1. Read the issue (`gh issue view <num>`) and inspect only the minimal files needed to understand scope.
2. Check `docs/DECISIONS.md` (if present) so the proposed scope does not contradict any locked Architectural Decision Record (ADR).
3. Ensure a type label (`bug` or `enhancement`) is applied.
4. **Oversized check:** If the prompt spans multiple unrelated modules or days of work, propose/create atomic child issues first so each `In Progress` ticket maps cleanly to one branch and one PR.
5. **If information or product decisions are missing:**
   - Apply the `need info` label (`gh issue edit <num> --add-label "need info"`).
   - Post a concise, numbered list of questions as an issue comment.
   - Leave the card in `Review` waiting for user input.
6. **If scope is clear and atomic:**
   - Rewrite/expand the issue body using the PO-Refined template in [references/issue-and-pr-templates.md](references/issue-and-pr-templates.md) (`User Story`, `Acceptance Criteria`, `Likely Files / Area`, and `Agent Metadata` containing `Target Role` and `Risk Tier: standard | high-risk`).
   - Apply `ready` and remove `need info` (`gh issue edit <num> --add-label "ready" --remove-label "need info"`).
   - Move the project card back to `Backlog`.

### Stage 3: `In Progress` (Subagent Execution & Lead Audit)
When a ticket moves to `In Progress`:
1. **Branch:** Never commit directly to `main`. Create `feat/<issue>-<slug>`, `fix/<issue>-<slug>`, or `chore/<issue>-<slug>`.
2. **Delegate Execution:** Launch the matching domain `flash` subagent (unless covered by the 1–2 line trivial-fix exception) with the issue's `User Story`, `Acceptance Criteria`, `Likely Files / Area`, relevant ADRs, and verification commands from `.agents/rules/kanban-workflow.md`.
3. **Subagent Deliverable:** The subagent implements the change, adds/updates tests, runs all required verification commands, commits (`<type>: <summary>`), pushes the branch, opens a Pull Request referencing `Closes #<num>`, and reports the PR URL and test output to the Lead.
4. **Lead Diff Audit:** The Lead inspects `gh pr diff <pr>` against the Acceptance Criteria, code quality, and ADRs (`DECISIONS.md`).
   - If changes are required, send targeted, actionable feedback back to the same `flash` subagent to push fix commits—do **not** burn Lead tokens rewriting bulk code directly.
5. **Conditional Adversarial Review:** If the ticket's `Risk Tier` is `high-risk` **or** `gh pr diff --name-only` touches any high-risk path defined in `.agents/rules/kanban-workflow.md` (e.g., auth, RLS, SQL migrations, privacy/visibility, billing/points math), invoke a read-only `sec-privacy` / `qa-tester` subagent to audit the diff before moving forward. Fix any confirmed issues via the Dev subagent.
6. **Promote:** Once checks and audits pass, post/update the PR summary with verification evidence and move the project card to `In Review`.

### Stage 4: `In Review` (PR & Preview Gate)
- **State:** PR is open with passing checks, audit summary, and Deploy Preview / build verification instructions ready for human testing.
- **Strict Human Merge Gate:** **NEVER** merge the PR or move the card to `Done` on your own. Stop and wait until:
  - The PR is explicitly approved on GitHub, **or**
  - The user writes the literal command **`approve and merge`**.

### Stage 5: `Done` (Squash Merge & Continuity Sync)
Once the human merge gate is satisfied:
1. Squash-merge the PR and delete the branch: `gh pr merge <pr> --squash --delete-branch`.
2. Move the GitHub Project card to `Done` (and verify `Closes #<num>` closed the issue).
3. Update `docs/PROGRESS.md` (and tick `docs/ROADMAP.md` if tracked).
4. For milestone releases, update `CHANGELOG.md` and create/push a SemVer tag (`v0.x.y`).

---

## 4. Quota-Resilient Session Continuity (`docs/` Triad)

Chat history can be interrupted or compacted at any time. Keep durable repository state in `docs/` (see [references/project-docs-templates.md](references/project-docs-templates.md)):

### Start of Every Session
1. Read `docs/PROGRESS.md` (if present) for current status, open PRs/branches, blockers, and the **exact next step**.
2. Read `docs/DECISIONS.md` (if present) before proposing any stack, schema, protocol, or design change.
3. Check `docs/ROADMAP.md` and/or query the requested GitHub Project column (`Review`, `In Progress`, `In Review`).

### End of Every Session or Meaningful Milestone
1. Prepend a short entry at the top of `docs/PROGRESS.md` and update `## Current status` with active branch/PR numbers and the **exact next step**.
2. Tick completed items in `docs/ROADMAP.md`.
3. Never silently change locked stack, protocol, or architecture choices—add a new ADR in `docs/DECISIONS.md` that explicitly supersedes the previous one.
4. Never leave uncommitted work stranded without noting its branch and status.

---

## 5. Token-Efficiency Principles

1. **Batch on demand:** Inspect the project board only when asked (e.g., *"process everything in Review"*). Never run background polling loops.
2. **Pre-compute context once in `Review`:** Recording `Likely Files / Area`, `Target Role`, `Risk Tier`, and `Acceptance Criteria` in the issue body prevents execution subagents from scanning the entire repository.
3. **Separate reasoning from typing:** Use `flash` for PO refinement, code generation, and test authoring; reserve high-reasoning Lead tokens for architecture checks, `gh pr diff` reviews, and merge decisions.
4. **Conditionally gate expensive audits:** Run the adversarial security/QA pass only when `Risk Tier: high-risk` or high-risk file globs are touched; validate standard tickets with automated typecheck, tests, and preview builds.
