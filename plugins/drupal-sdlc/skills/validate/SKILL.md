---
name: validate
description: "Run full validation on recent changes including linting, config import, config round-trip check, Playwright screenshots, and tests. Use after completing a feature, when asked to validate work, or when verifying that changes are correct."
---

# Validate

Run comprehensive validation on recent work with visual verification.

## Before You Start

Check for a project-level iteration log:
```bash
cat .claude/skills/validate/iterations.md 2>/dev/null
```
If it exists, read it and apply any relevant learnings to this run.

Also read the plugin-level learnings in the `## Iteration Log` section at the bottom of this file.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml — will use defaults"`
```

## Changed Files
```
!`git diff --name-only HEAD 2>/dev/null; git diff --name-only --cached 2>/dev/null; git ls-files --others --exclude-standard 2>/dev/null`
```

## Workflow

### 1. Categorize changed files

From the changed files above, categorize: PHP files, YAML files, Twig files, config YAML (`config/sync/` or `config/install/`), Playwright tests, other.

### 2. Lint PHP files

For each changed `.php`, `.module`, `.install`, `.inc`, `.theme`, `.test` file:
- Run `php -l {file}` for quick syntax check
- If all pass, run the full quality command from drupal-claude.yml
- Report any failures with the error output

### 3. Lint YAML files

For each changed `.yml` file:
- Verify valid YAML syntax
- For config/sync YAML: verify no `uuid` or `_core` keys were accidentally added

### 4. Import config (if config YAML changed)

If any files in `config/sync/`, `config/install/`, or `config/split/` were changed:
- Run: `{drush_prefix} config:import -y`
- Report any import errors
- Follow up with: `{drush_prefix} cache:rebuild`
- Run `{drush_prefix} config:status` to confirm no differences remain

### 5. Config round-trip check (if config YAML changed)

After config:import, verify the config round-trips cleanly:
- Run: `{drush_prefix} config:export -y`
- Run: `git diff --exit-code config/sync/`
- If diff exists: **WARNING** — committed config doesn't match what Drupal exports. This means the hand-written config differs from what Drupal produces. The exported version is authoritative — commit it instead.

### 6. Visual verification (if UI-affecting changes)

If the changes affect UI (new fields, form displays, view displays, blocks, templates, theme changes):
- Identify the URL(s) where changes would be visible
- Use Playwright MCP to navigate to each URL
- Log in as admin if the page requires authentication
- Take a full-page screenshot of each affected page
- **Always save screenshots to `tests/playwright/test-results/screenshots/`**
  Use filename format: `{feature-name}-{date}.png`
- Share the screenshot and confirm it looks correct before proceeding

Types of changes that need visual verification:
- New/modified config YAML for form or view displays
- New/modified Twig templates
- New/modified blocks or views
- New/modified CSS or theme changes
- New module with a config form

### 7. Run Playwright tests

**Always run this step if any Playwright tests exist.**
```bash
cd tests/playwright && npx playwright test --reporter=list 2>&1
```

If tests fail:
- Read the full failure output carefully
- Identify root cause — is it a code issue, config issue, or test issue?
- Fix the underlying problem in the codebase or config
- Re-run the full Playwright suite
- Repeat until ALL tests pass
- Never proceed to PR creation if any Playwright test is failing

If no Playwright tests exist for the affected area:
- Note this gap explicitly
- Use the test-writer skill to generate tests before proceeding
- Run the newly generated tests and confirm they pass

### 8. Run PHPUnit tests (if PHP files changed)

If any `.php`, `.module`, or `.install` files were changed or created:
```bash
ddev phpunit web/modules/custom 2>&1
```

If tests fail:
- Read the failure output carefully
- Fix the underlying issue
- Re-run until all tests pass
- Never proceed to PR creation if any PHPUnit test is failing

If no PHPUnit tests exist for the changed PHP:
- Note this gap explicitly
- Use the test-writer skill to generate unit tests before proceeding

### 9. Summary

Present a validation checklist:
```
Validation Results:
- [ ] PHP syntax:            {pass/fail/skipped}
- [ ] Quality checker:       {pass/fail/skipped}
- [ ] YAML syntax:           {pass/fail/skipped}
- [ ] Config import:         {pass/fail/skipped}
- [ ] Config round-trip:     {pass/fail/skipped}
- [ ] Config status clean:   {pass/fail/skipped}
- [ ] Screenshots captured:  {yes/no/not needed}
- [ ] Playwright tests:      {pass/fail/none — generated and passed/skipped}
- [ ] PHPUnit tests:         {pass/fail/none — generated and passed/skipped}
```

**Only ask the user to review if ALL checks are passing.**
If anything is failing, fix it first — do not present a failed summary and ask for confirmation.

Ask the user: *"All checks are passing. Please review the results above and the screenshots. Can you confirm everything is working as expected before I raise the PR?"*

## Visual Verification Reference

| Change Type | Visual Verification? | What to Screenshot |
|---|---|---|
| New field added | Yes | Form display + view display |
| Config form created | Yes | The admin config form page |
| Block placed | Yes | The page where block appears |
| View created/modified | Yes | The view page output |
| Template changed | Yes | Pages using that template |
| Theme changes | Yes | Homepage + affected pages |
| PHP service logic only | No | — |
| API client changes | No | — |
| Test files only | No | — |
| Quality/tooling only | No | — |

## Important Rules

- **Never proceed to PR if any test is failing**
- **Never skip Playwright tests if they exist**
- **Always fix failures before presenting the summary**
- If DDEV is not running, warn the user — config import and visual verification require it
- Screenshots must use full page capture
- If Playwright MCP is not available, instruct the user to manually verify the URLs

## Iteration Log

Record learnings here after real uses that reveal non-obvious validation sequencing or environment issues.
Format: `[YYYY-MM-DD] <lesson>` — keep entries short and actionable.

_No entries yet. Add the first one after the first real use._
