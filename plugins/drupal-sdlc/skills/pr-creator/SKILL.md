---
name: pr-creator
description: "Create a pull request from completed work. Generates PR title, description, what changed, test plan, and risk level. Always run the validate skill before using this."
---

# PR Creator

Generate a complete pull request from finished work.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml found"`
```

## Current Branch
```
!`git branch --show-current 2>/dev/null`
```

## Changed Files vs Main
```
!`git diff --name-only main 2>/dev/null || git diff --name-only HEAD~1`
```

## Full Diff
```
!`git diff main 2>/dev/null || git diff HEAD~1`
```

## Instructions

1. Read the diff and changed files above
2. Read drupal-claude.yml to get `github.owner`, `github.repo`, `jira.cloud_id`, and `jira.project_key`
3. Extract the Jira ticket key from the current branch name
   (e.g. feat/TDP-6-category-tags → TDP-6)
4. Generate the following:

---

**PR Title:**
Use conventional commit format with Jira key: `feat({KEY}-X): short description`

**PR Description:**

## What this PR does
2-3 sentences explaining the change in plain English.

## Why
Why this change was needed — reference the Jira ticket.

## Changes
List every file created or modified with one line explaining why:
- `path/to/file.yml` — reason
- `path/to/file.php` — reason

## Test Plan
How to manually verify this works:
1. Step by step instructions
2. What to look for

## Automated Tests
- Which Playwright tests cover this
- Which PHPUnit tests cover this

## Risk Level
Low / Medium / High — reason

Resolves {KEY}-X
Jira: https://{jira.cloud_id}/browse/{KEY}-X

---

5. Use GitHub MCP to create the PR:
   - owner: from drupal-claude.yml `github.owner`
   - repo: from drupal-claude.yml `github.repo`
   - title: feat({KEY}-X): short description
   - body: the PR description above
   - head: current branch
   - base: main

6. Confirm the PR URL with the user.

## Important Rules
- Always reference the Jira ticket with "Resolves {KEY}-X" and the Jira link
- Never raise a PR without tests existing or a documented manual test plan
- Always use conventional commit format with Jira key: feat({KEY}-X):
- Never include sensitive data in PR descriptions
- Never add attribution stamps to PR descriptions
- Never use gh CLI commands — always use GitHub MCP tools instead
