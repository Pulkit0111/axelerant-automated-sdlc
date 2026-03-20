---
name: hotfix
description: "Urgent fix bypass workflow: read ticket, branch, fix, validate, PR. Skips spec-writer for speed. Use when told to hotfix a Jira ticket or for urgent bug fixes."
argument-hint: "<Jira ticket key e.g. TDP-4>"
---

# Hotfix

Streamlined workflow for urgent fixes. Skips spec generation for speed.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null`
```

## Instructions

You are running a streamlined hotfix workflow for an urgent fix.
This skips the spec-writer step but still follows quality and testing standards.

---

### Step 1 — Read the Jira ticket

Use Atlassian MCP getJiraIssue to read ticket $ARGUMENTS.
Extract: summary, description, reproduction steps, expected behavior.

**On Failure:** Present the error to the user. Do NOT proceed.

---

### Step 2 — Create hotfix branch

Switch to main and pull latest:
```bash
git checkout main && git pull origin main
```

Create a new git branch:
`hotfix/$ARGUMENTS-short-description`

Where short-description is 3-4 words from the ticket summary in kebab-case.

---

### Step 3 — Transition ticket to In Progress

Use Atlassian MCP getTransitionsForJiraIssue to find the "In Progress" transition ID,
then use transitionJiraIssue to move $ARGUMENTS to "In Progress".

---

### Step 4 — Implement the fix

Diagnose the issue and implement the fix.

After every file change:
- Run the quality command from drupal-claude.yml
- Fix any failures before continuing

For config YAML changes always run in this exact order:
1. {drush_prefix} config:import -y
2. {drush_prefix} config:export -y
3. {drush_prefix} config:status (must show "No differences")

---

### Step 5 — Validate

Run the validate skill. Everything must pass before moving on.
If anything fails — fix it, re-run validation. Repeat until clean.

---

### Step 6 — Run tests

Run existing Playwright and PHPUnit tests to ensure no regressions.
If needed, add a regression test for the specific bug being fixed.

```bash
cd tests/playwright && npx playwright test --reporter=list 2>&1
```

NEVER proceed if any new test failures were introduced.

---

### Step 7 — Create PR

Use GitHub MCP to create the PR.

**PR title format:** `fix({KEY}-X): short description of fix`

**PR body must include:**
- What was broken
- Root cause
- What was fixed
- Reference: `Fixes {KEY}-X`
- Jira link
- Test results
- Risk level

---

### Step 8 — Post PR link to Jira

Use Atlassian MCP addCommentToJiraIssue to post the PR URL on the ticket.

---

### Step 9 — Notify user

Tell the user:
- PR URL
- Jira ticket URL
- Test results
- Risk level
- "Hotfix PR is ready for your review and merge."

## Hard Rules

- Never skip validation (Step 5)
- Never skip tests (Step 6)
- Never modify protected files without explicit user approval
- Never write UUID values in config YAML
- Always post PR link back to Jira ticket
- Never add attribution stamps to PR descriptions or Jira comments
- Never use gh CLI commands — always use GitHub MCP tools instead
