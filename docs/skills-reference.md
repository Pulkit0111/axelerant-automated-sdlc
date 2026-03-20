# Skills Reference

## work-on-jira-ticket

**Trigger:** `work on jira ticket {KEY}-X` or `/work-on-jira-ticket {KEY}-X`

**What it does:** Runs the complete SDLC loop autonomously — reads the Jira ticket, generates a spec, builds the feature, validates, tests, creates a PR, reviews it, and posts results back to Jira.

**Inputs:** Jira ticket key (e.g. TDP-4)

**Outputs:** PR URL, Jira comment with spec, Jira comment with PR link, review comment on PR

**Human Checkpoints:**
- After spec generation — must approve before building
- After PR creation — must review and merge manually

**Failure Modes:**
- Jira ticket not found → stops, asks user
- Config import fails → posts error to Jira, stops
- Tests fail → fixes and retries, stops if unfixable

---

## spec-writer

**Trigger:** `/spec-writer <description>` or called by work-on-jira-ticket

**What it does:** Translates plain English into a structured Drupal implementation spec with numbered acceptance criteria mapped to test files.

**Inputs:** Plain English feature description or Jira ticket content

**Outputs:** Structured spec with: intent, implementation plan, numbered ACs with test mapping, file list, risk level

**Key feature:** Names specific Drupal constructs — content types, field names, views, blocks, permissions

---

## config-builder

**Trigger:** `/config-builder <description>`

**What it does:** Generates Drupal config YAML files (field storage, field instance, form display, view display) from natural language.

**Inputs:** Description of field or config needed

**Outputs:** Config YAML files in `config/sync/`, imported and exported through Drush

**Key rule:** Never writes UUIDs — always runs config:import + config:export cycle

---

## module-scaffolder

**Trigger:** `/module-scaffolder <module name and description>`

**What it does:** Generates complete custom module boilerplate — info.yml, services, routing, forms, API client, Drush commands.

**Inputs:** Module machine name, description, API details

**Outputs:** Complete module directory under `web/modules/custom/`

---

## test-writer

**Trigger:** `/test-writer <what to test>`

**What it does:** Generates Playwright specs and PHPUnit tests following established patterns.

**Inputs:** Feature, module, or component to test

**Outputs:** Test files in `tests/playwright/tests/` and/or `web/modules/custom/{module}/tests/`

**Key features:**
- Multi-role test scaffolding (anonymous, authenticated, admin)
- Mandatory tags on every test (@smoke, @fast, @readOnly, etc.)
- Always uses shared login helpers

---

## validate

**Trigger:** `/validate` or called by work-on-jira-ticket

**What it does:** Runs comprehensive validation: PHP lint, quality checker, YAML validation, config import, config round-trip check, visual verification, Playwright tests, PHPUnit tests.

**Inputs:** None — works on current changes

**Outputs:** Validation checklist with pass/fail for each check, screenshots

**Key feature:** Config round-trip check — verifies that config:export produces identical output to committed config

---

## code-quality-fixer

**Trigger:** `/code-quality-fixer <error output>`

**What it does:** Fixes PHPStan, PHPCS, twigcs, and yamllint violations from error output.

**Inputs:** Linter error output or file paths

**Outputs:** Fixed files, re-runs quality command to verify

---

## pr-creator

**Trigger:** `/pr-creator` or called by work-on-jira-ticket

**What it does:** Creates a GitHub PR with full context — description, file changes, test results, risk level, Jira reference.

**Inputs:** None — reads current branch diff

**Outputs:** GitHub PR (created via MCP), PR URL

---

## pr-reviewer

**Trigger:** `/pr-reviewer <PR number>` or called by work-on-jira-ticket

**What it does:** Reviews a PR diff, runs security checklist, checks AC coverage, and posts findings as a GitHub comment.

**Inputs:** PR number (optional — defaults to current branch's PR)

**Outputs:** Review comment posted on GitHub PR

**Key features:**
- Security checklist (SQL injection, XSS, hardcoded credentials, route permissions, etc.)
- AC coverage matrix — maps spec ACs to actual tests

---

## hotfix

**Trigger:** `/hotfix {KEY}-X`

**What it does:** Streamlined urgent fix workflow — reads ticket, branches, fixes, validates, creates PR. Skips spec-writer for speed.

**Inputs:** Jira ticket key

**Outputs:** Hotfix PR with `fix({KEY}-X):` prefix

**Key difference from work-on-jira-ticket:** No spec generation, no human approval gate before building — optimized for speed on urgent fixes.
