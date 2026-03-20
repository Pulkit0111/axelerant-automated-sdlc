---
name: work-on-jira-ticket
description: "Pick up a Jira ticket and run the complete SDLC loop autonomously. Use when told to work on a Jira ticket key like TDP-4 or PROJ-12."
argument-hint: "<Jira ticket key e.g. TDP-4>"
---

# Work On Jira Ticket

Run the complete AI-native SDLC loop for a Jira ticket.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null`
```

## Project Config Values
```
!`cat drupal-claude.yml 2>/dev/null | grep -E '(owner|repo|cloud_id|project_key|board_url|drush_prefix|quality_command|base_url):' || echo "drupal-claude.yml not found — ask user for project details"`
```

## Instructions

You are running a complete autonomous SDLC loop driven by a Jira ticket.
Follow every step in order. Do not skip steps.
Do not move to the next step without completing the current one fully.

Read `drupal-claude.yml` to get all project-specific values:
- GitHub owner/repo from `github.owner` and `github.repo`
- Jira cloud ID from `jira.cloud_id`
- Jira project key from `jira.project_key`
- Drush prefix from `local_dev.drush_prefix`
- Quality command from `local_dev.quality_command`

---

### Step 1 — Read the Jira ticket

Use Atlassian MCP getJiraIssue to read ticket $ARGUMENTS from the Jira cloud ID in drupal-claude.yml.

Also read:
- Parent epic if exists — for broader context
- Any subtasks — to understand the full scope
- All existing comments — to understand prior decisions
- Any linked issues — to understand dependencies and blockers

Extract: summary, description, acceptance criteria, dependencies.

If the ticket has subtasks, read each subtask as well.
If the ticket is blocked by another ticket, warn the user before proceeding.

**On Failure:** Post error context as a Jira comment and present the error to the user. Do NOT proceed.

---

### Step 2 — Generate spec (HUMAN CHECKPOINT)

Use the spec-writer skill to generate a complete implementation plan
based on the Jira ticket content.

The spec must include:
- Intent (one sentence)
- What already exists vs what needs to be built
- Drupal Implementation Plan — naming specific content types, field names + types, views, blocks, templates, permissions
- Acceptance Criteria (numbered, from the Jira ticket ACs) with test mapping:
  | AC | Test File | Test Name |
  |---|---|---|
  | AC-1 | tests/playwright/tests/feature.spec.ts | should display X @smoke |
- Files to Create/Modify (table format)
- Test Plan (Playwright + PHPUnit)
- Risk Level (Low/Medium/High) with reason
- Open Questions (if any)

**STOP HERE.** Present the spec to the user and WAIT for explicit approval.
Say: "Spec is ready. Please review and say 'approved' or 'go ahead' to proceed."
Do NOT proceed until the user explicitly approves.

**On Failure:** Present the error to the user and ask for guidance.

---

### Step 3 — Post spec to Jira ticket

Use Atlassian MCP addCommentToJiraIssue to post the approved spec
as a comment on ticket $ARGUMENTS.

Format the comment with clear markdown sections:
## Implementation Spec — {ticket summary}

### Intent
{one sentence}

### Implementation Plan
{plan details with specific Drupal constructs named}

### Acceptance Criteria
- [ ] AC-1: criterion 1
- [ ] AC-2: criterion 2

### Files to Create/Modify
| File | Action |
|---|---|
| path/to/file | Create/Modify — reason |

### Test Plan
- Playwright: {what to test}
- PHPUnit: {what to test or "None — config only"}

### Risk Level
{Low/Medium/High} — {reason}

**On Failure:** Warn the user that the spec could not be posted to Jira. Continue with the build.

---

### Step 4 — Transition ticket to In Progress

Use Atlassian MCP getTransitionsForJiraIssue to find the "In Progress" transition ID,
then use transitionJiraIssue to move $ARGUMENTS to "In Progress".

**On Failure:** Warn the user and continue — the transition is non-blocking.

---

### Step 5 — Create branch

First switch to main and pull latest:
```bash
git checkout main && git pull origin main
```

Create a new git branch:
feat/$ARGUMENTS-short-description

Where short-description is 3-4 words from the ticket summary in kebab-case.

---

### Step 6 — Implement

Use the appropriate skills based on what the spec requires:
- Config only → config-builder skill
- New module → module-scaffolder skill
- Code quality issues → code-quality-fixer skill
- Mix of config + code → use both skills in sequence

After every file change:
- Run the quality command from drupal-claude.yml
- Fix any failures before continuing

For config YAML changes always run in this exact order:
1. {drush_prefix} config:import -y
2. {drush_prefix} config:export -y
3. {drush_prefix} config:status (must show "No differences" before proceeding)

Always commit the post-export files — never the hand-written originals.
Never write UUID values in config YAML — let config:export generate them.

If the ticket has subtasks:
- Work through each subtask in order
- Comment on each subtask in Jira as it is completed

**On Failure:** Post error context to Jira as a comment. Present the error to the user. Do NOT proceed.

---

### Step 7 — Validate

Run the validate skill. Everything must pass before moving on.
If anything fails — fix it, re-run validation. Repeat until clean.

---

### Step 8 — Run ALL tests

**Playwright tests:**
```bash
cd tests/playwright && npx playwright test --reporter=list 2>&1
```
- If tests fail: diagnose root cause, fix, re-run. Repeat until ALL pass.
- If no tests exist: use test-writer skill to generate them first, then run.
- ALWAYS use loginAsAdmin() from tests/playwright/helpers/auth.ts
- NEVER write inline login code in test files
- Use Playwright MCP to visually verify UI changes in the browser
- Save screenshots to: tests/playwright/test-results/screenshots/

**PHPUnit tests (if PHP was written):**
```bash
{drush_prefix_without_drush} phpunit web/modules/custom 2>&1
```
- If tests fail: fix and re-run until all pass.
- If no tests exist: use test-writer skill to generate them first.

**Pre-existing failures:**
- Use git stash to verify which failures existed before your changes
- Only fix failures introduced by the current ticket
- Document any pre-existing failures in the PR description

NEVER proceed to Step 9 if any new test failures were introduced.

---

### Step 9 — Raise PR

Use pr-creator skill to generate the PR content.
Use GitHub MCP to create the PR on {github.owner}/{github.repo} from drupal-claude.yml.

**PR title format:** `feat({KEY}-X): short description`

**PR body must include:**
- What this PR does (2-3 sentences)
- Why it was needed
- Reference: `Resolves {KEY}-X`
- Jira link: `https://{jira.cloud_id}/browse/{KEY}-X`
- Files changed (table)
- Test results (X/X passing, note pre-existing failures)
- Manual test plan
- Risk level

---

### Step 10 — Review PR

Use pr-reviewer skill to review the PR.
The skill will:
1. Analyze the PR diff using full project context
2. Run the security checklist
3. Fix any blocking issues found in the codebase
4. Commit and push fixes if needed
5. Re-run quality checks and tests after any fixes
6. Post the complete review as a comment on the PR via GitHub MCP
7. Confirm the review comment URL

Do not proceed to Step 11 until the review comment is confirmed posted.

---

### Step 11 — Post PR link back to Jira ticket (HUMAN CHECKPOINT)

Use Atlassian MCP addCommentToJiraIssue to post the PR URL
as a comment on ticket $ARGUMENTS.

Post this comment:

## PR Raised

**PR:** https://github.com/{github.owner}/{github.repo}/pull/{number}
**Branch:** feat/$ARGUMENTS-short-description
**Status:** Ready for final review and merge

**Test Results:** X/X passing
**Risk Level:** Low/Medium/High
**Files Changed:** X files

**What was built:**
{2-3 sentence summary of what was implemented}

---

### Step 12 — Notify user

Tell the user:
- PR URL
- Jira ticket URL
- Review comment URL
- Test results (X/X passing)
- Risk level
- Any subtasks still remaining
- Any follow-up tickets that should be created in Jira
- "Ready for your final review and merge. I will NOT merge this PR — that's your call."

---

### Post-Merge Step (when user confirms PR is merged)

When the user says the PR has been merged:

1. Note: The GitHub Action automatically transitions the Jira ticket to Done.
   Only manually transition if the user asks you to.

2. Switch to main and pull latest:
   ```bash
   git checkout main && git pull origin main
   ```

3. Tell the user: "Switched to main and pulled latest. The GitHub Action will transition the Jira ticket to Done automatically. Ready for the next ticket."

---

## Hard Rules

- Never skip the spec approval step (Step 2) — HUMAN CHECKPOINT
- Never raise a PR with new failing tests (Step 8 must be clean)
- Never modify protected files without explicit user approval
- Always post spec as a comment on the Jira ticket (Step 3)
- Always transition ticket to In Progress before building (Step 4)
- Always run config:export after config:import (Step 6)
- Never write UUID values in config YAML
- Always use loginAsAdmin() from the shared helper in Playwright tests
- Never write inline login code in test files
- Always save screenshots to tests/playwright/test-results/screenshots/
- Always post PR link back to Jira ticket (Step 11)
- If blocked by something requiring human input — stop and ask clearly
- Never add attribution stamps to PR descriptions, review comments, or Jira comments
- Never use gh CLI commands — always use GitHub MCP tools instead
- On any step failure: post error context to Jira, present error to user, do NOT proceed
