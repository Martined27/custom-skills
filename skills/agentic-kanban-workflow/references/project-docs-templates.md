# Repo-Native Session Continuity Templates (`AGENTS.md` & `docs/`)

Use these templates to bootstrap **Repo-Docs Mode** or **Hybrid Mode** so any agent can resume work after context compaction or quota resets without relying on chat history.

---

## 1. `AGENTS.md` (Repository Entry Point)

```markdown
# <PROJECT_NAME>: Agent Instructions

Work may be interrupted at any time (quota limits or session resets), so **repository files and GitHub issues/PRs are the single source of truth—never chat history.**

## Start of Every Session
1. Read `docs/PROGRESS.md` (current status, open PRs, exact next step).
2. Read `docs/DECISIONS.md` before proposing any stack, schema, protocol, or architecture change.
3. Check `docs/ROADMAP.md` and/or the GitHub Kanban board (`.agents/rules/kanban-workflow.md`).

## End of Every Session (or Meaningful Milestone)
1. Prepend an entry at the top of `docs/PROGRESS.md` and update `## Current status` with the **exact next step**.
2. Tick completed items in `docs/ROADMAP.md`.
3. Never leave unpushed local-only changes undocumented; ensure work is committed on a branch with an open PR or noted in `docs/PROGRESS.md`.

## Git, PR & Subagent Rules
- Never commit directly to `main`. Use `feat/<issue>-<slug>`, `fix/<issue>-<slug>`, or `chore/<issue>-<slug>`.
- Delegate implementation work to role-scoped `flash` subagents (`designer`, `backend-dev`, `mobile-dev`, `qa-tester`, `sec-privacy`) except trivial 1–2 line fixes.
- Lead agent audits `gh pr diff`, sends revision notes back to the subagent if fixes are needed, and waits for explicit human approval (`approve and merge` or GitHub approval) before squash-merging (`gh pr merge --squash --delete-branch`).
```

---

## 2. `docs/PROGRESS.md`

```markdown
# <PROJECT_NAME>: Progress & Handoff Log

> **Read this first in every new session.** Update it before ending every session or meaningful milestone.
> Newest entry on top. Keep entries factual: what was done, what passed, active PRs/branches, blockers, and exact next step.

---

## Current status
- **Phase / Milestone:** <e.g., MVP Sprint 2>
- **Active Branch / PR:** <none | `feat/12-auth-guard` (PR #14 in `In Review`)>
- **Next step:** <Exact next action for the next session>
- **Blockers:** <none | description>

---

## YYYY-MM-DD: Session <N>
- Completed `<PR # / Issue #>` on branch `<branch-name>`:
  - <Key change 1>
  - <Key change 2>
- **Verification:** `<exact commands run and pass counts>`
- **Next:** <Exact next action>
```

---

## 3. `docs/DECISIONS.md` (Architectural Decision Records)

```markdown
# <PROJECT_NAME>: Decision Log (ADR)

> Locked architectural and product decisions live here. **Never silently change a locked decision—append a new ADR that explicitly supersedes the prior one.**

---

## ADR-001: <Short Decision Title>
- **Date:** YYYY-MM-DD · **Status:** Accepted
- **Decision:** <What was chosen>
- **Why:** <Trade-offs and rationale>
- **Consequences:** <Rules agents and code must obey going forward>
```

---

## 4. `docs/ROADMAP.md`

```markdown
# <PROJECT_NAME>: Roadmap & Checklist

Legend: `[ ]` todo · `[/]` in progress · `[x]` done. Keep this file in sync with `docs/PROGRESS.md` and closed Kanban issues.

## Milestone 1: <Name>
- [x] <Completed item> (PR #1)
- [/] <Active item> (`In Progress` — Issue #12 / Branch `feat/12-...`)
- [ ] <Upcoming item>
```
