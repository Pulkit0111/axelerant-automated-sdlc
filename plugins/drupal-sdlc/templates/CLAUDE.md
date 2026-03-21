<!--
⚠️  SETUP REQUIRED — Read this before using Claude Code on this project

This file tells Claude everything it needs to know about your project.
It is loaded automatically at the start of every Claude Code session.

Before you start:
1. Replace "Project Name" in the heading below with your actual project name
2. Fill in ALL sections marked with ⚠️ REQUIRED
3. Update the Workflow section if your branching or drush conventions differ
4. Save the file — Claude will read it on next session start

Incomplete sections = Claude working with wrong project context = wrong branches, wrong Jira keys, wrong repos.
-->

# Project Name — Claude Code Guide

Project-specific configuration is in `drupal-claude.yml`. Read it at the start of every task.

## Project Overview
<!-- ⚠️ REQUIRED: Replace this comment with 2-3 sentences describing your project.
     Example: "Drupal 11 e-commerce platform for Acme Corp. Uses Commerce, Paragraphs,
     and a custom Salesforce integration. Primary Jira project: ACME." -->

## Jira Project
<!-- ⚠️ REQUIRED: Fill in your Jira details. Claude uses these to read tickets and post comments.
     Remove this section entirely if your project doesn't use Jira. -->
- Project: <!-- e.g. My Drupal Project -->
- Key: <!-- e.g. MDP -->
- Site: <!-- e.g. mycompany.atlassian.net -->
- Board: <!-- e.g. https://mycompany.atlassian.net/jira/software/projects/MDP/boards/1 -->

## GitHub Repo
<!-- ⚠️ REQUIRED: Fill in your GitHub details. Claude uses these to create and review PRs.
     These must match the values in drupal-claude.yml → github.owner and github.repo. -->
- Owner: <!-- e.g. my-org or my-username -->
- Repo: <!-- e.g. my-drupal-project -->
- URL: <!-- e.g. https://github.com/my-org/my-drupal-project -->

## Workflow — Follow This Every Time
1. Read the Jira ticket using Atlassian MCP
2. Use the spec-writer skill to generate the implementation plan
3. Post the spec as a comment on the Jira ticket
4. Transition the ticket to In Progress
5. Switch to main and pull latest: `git checkout main && git pull origin main`
6. Create a branch: feat/{KEY}-X-short-description
7. Build using config-builder and/or module-scaffolder skills
8. Run the validate skill after every change
9. Use test-writer to generate and run tests
10. Use pr-creator to raise the PR
11. Use pr-reviewer to review and post review comment on PR
12. Post PR link back to Jira ticket
13. Notify user — ready for final review and merge
14. After merge — GitHub Action automatically transitions Jira ticket to Done

## Trigger Command
To work on a Jira ticket type:
work on jira ticket {KEY}-X

## Coding Standards
- Follow Drupal coding standards
- All code must pass quality checker before commit
- Services use constructor injection via *.services.yml
- Never call \Drupal::service() in classes
- Config forms extend ConfigFormBase

## AI Guardrails
- NEVER modify any file not explicitly required by the current task
- Always show a diff before making changes
- Always state risk level: Low / Medium / High
- Ask before touching more than one module at a time
- NEVER write UUID values in config YAML — let config:export generate them

## Protected Files — NEVER modify without explicit approval
- web/sites/default/settings.php
- web/sites/default/settings.ddev.php
- *.install files
- config_split.config_split.*.yml

## Quality Checks
After every change run the quality command from `drupal-claude.yml`.
Check that all PHP, YAML, Twig, and JSON files are valid before committing.

## Config Management — ALWAYS follow this order
1. {drush_prefix} config:import -y
2. {drush_prefix} config:export -y
3. {drush_prefix} config:status (must show "No differences")
Always commit post-export files — never hand-written config YAML.

## Known Pitfalls — Read Before Every Task

### 1. Never hand-craft UUIDs in config YAML
Leave the uuid key absent. config:export adds the correct UUID after import.

### 2. Never write inline login in Playwright tests
Always use loginAsAdmin() from tests/playwright/helpers/auth.ts.

### 3. Always run config:export after config:import
Never commit hand-written config YAML. Always commit the post-export version.

### 4. Never save screenshots to the project root
Always save to tests/playwright/test-results/screenshots/

### 5. Never use #edit-submit selector in Playwright tests
Use getByRole('button', { name: 'Save' }) instead.

### 6. CKEditor body field
Never use #edit-body-0-value — it is hidden by CKEditor5.
Always use page.getByRole('textbox', { name: 'Rich Text Editor' })

### 7. Page title block
The page title block must be enabled and in the content region.
Without it, certain pages will have no h1.

### 8. Always post PR link back to Jira
After raising a PR, always comment the PR URL on the Jira ticket.

### 9. Jira ticket transitions to Done automatically
A GitHub Action handles the Done transition when the PR is merged. Do NOT manually transition to Done.

### 10. Never use gh CLI — use GitHub MCP instead
Always use GitHub MCP tools for creating PRs, posting reviews,
and reading PR diffs. Never use gh pr create, gh pr diff,
or gh issue list commands.

### 11. Never add attribution stamps
Never add "Generated with Claude Code" or "Reviewed with Claude Code"
or any similar stamps to PR descriptions, review comments, or Jira comments.
