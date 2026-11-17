# GitHub CLI (`gh`) Playbook for Agentic Kanban

Concrete commands for querying boards, running the Product Owner pass, moving cards across columns, auditing PR diffs, and completing the merge gate.

Replace `<OWNER>`, `<REPO>`, `<PROJECT_NUMBER>`, `<PROJECT_ID>`, `<STATUS_FIELD_ID>`, and `<OPTION_ID>` with the values from the repo's `.agents/rules/kanban-workflow.md`.

---

## 1. Discover Project & Column Option IDs (One-Time Setup)

```bash
# List projects for the user/org to get <PROJECT_NUMBER> and <PROJECT_ID>
gh project list --owner <OWNER> --format json

# Inspect fields and single-select Status option IDs
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Look for the field named `Status` (`PVTSSF_...`) and record the `id` for each option (`Backlog`, `Review`, `In Progress`, `In Review`, `Done`).

---

## 2. Inspect Board Columns & Issues On Demand

```bash
# List all project items with their item ID, status column, issue number, title, and labels
gh project item-list <PROJECT_NUMBER> --owner <OWNER> --limit 100 --format json

# Quick filter with jq for a specific column (e.g., "Review" or "In Progress")
gh project item-list <PROJECT_NUMBER> --owner <OWNER> --limit 100 --format json \
  | jq '.items[] | select(.status == "Review") | {item_id: .id, number: .content.number, title: .title, labels: .labels}'

# Read full issue details and comments before scoping or implementing
gh issue view <ISSUE_NUMBER> --repo <OWNER>/<REPO> --comments
```

---

## 3. Product Owner Pass (`Review`)

```bash
# Ensure type label is present
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --add-label "enhancement"

# Option A: Missing context -> tag `need info` and post numbered questions (keep in Review)
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --add-label "need info"
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body-file /tmp/need-info.md

# Option B: Clear scope -> rewrite body from file, tag `ready`, remove `need info`, move to Backlog
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body-file /tmp/refined-issue.md \
  --add-label "ready" --remove-label "need info"

gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_ID> \
  --field-id <STATUS_FIELD_ID> \
  --single-select-option-id <BACKLOG_OPTION_ID>
```

---

## 4. Dev Execution, Lead Diff Audit & `In Review` Promotion

```bash
# 1. Subagent creates branch from up-to-date main
git fetch origin && git checkout -b feat/<ISSUE_NUMBER>-<slug> origin/main

# 2. Subagent commits, pushes, and opens PR linking the issue
git add -A && git commit -m "feat: <concise summary>"
git push -u origin feat/<ISSUE_NUMBER>-<slug>
gh pr create --repo <OWNER>/<REPO> --base main --head feat/<ISSUE_NUMBER>-<slug> \
  --title "<type>: <concise title>" --body-file /tmp/pr-body.md

# 3. Lead Architect audits changed files and full diff
gh pr diff <PR_NUMBER> --repo <OWNER>/<REPO> --name-only
gh pr diff <PR_NUMBER> --repo <OWNER>/<REPO>

# 4. Move card from In Progress -> In Review once checks + Lead audit (+ adversarial pass if high-risk) pass
gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_ID> \
  --field-id <STATUS_FIELD_ID> \
  --single-select-option-id <IN_REVIEW_OPTION_ID>
```

---

## 5. Human Merge Gate & `Done`

Only execute after GitHub PR approval or the user's literal **`approve and merge`** instruction:

```bash
# Squash-merge and delete remote/local branch
gh pr merge <PR_NUMBER> --repo <OWNER>/<REPO> --squash --delete-branch

# Move card to Done
gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_ID> \
  --field-id <STATUS_FIELD_ID> \
  --single-select-option-id <DONE_OPTION_ID>

# Optional: tag milestone release after updating CHANGELOG.md and docs/PROGRESS.md
git checkout main && git pull origin main
git tag -a v0.x.y -m "Release v0.x.y"
git push origin v0.x.y
```
