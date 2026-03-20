---
name: module-scaffolder
description: "Generate complete Drupal custom module boilerplate, especially for API integrations. Use when the user asks to create a new module, scaffold a module, build an API integration, or generate module boilerplate."
argument-hint: "<module name and description>"
---

# Module Scaffolder

Generate custom Drupal modules following established patterns.

## Project Context

!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml found — ask user for local dev tool and drush prefix"`

## User's Request

$ARGUMENTS

## Workflow

1. Gather inputs: module machine name, human name, description, API name, required functionality
2. Generate directory at web/modules/custom/{module_name}/
3. Generate files using templates from module-template.md:
   - {module}.info.yml, {module}.services.yml, {module}.routing.yml, {module}.links.menu.yml
   - src/{Api}Client.php — API wrapper with @see docblock tags
   - src/Form/SettingsForm.php — config form
   - config/schema/{module}.schema.yml
   - config/install/{module}.settings.yml
4. Add Drush command if CLI operations needed (drush.services.yml + Commands class)
5. Add batch processing if bulk operations needed (src/Batch/{Operation}BatchManager.php)
6. Post-generation checklist: enable module, test config form, run quality checker

## Common Pitfalls

- Entity field length: truncate to schema max (e.g. mb_substr($value, 0, 60)) before setting fields
- Views bundle filter: use `operator: in` not `operator: '='` to avoid validation errors
- Config install vs sync: export config after first enable if module ships optional config (views, blocks)

## Validation and acceptance

Do not treat scaffold as done until:
1. At least one Playwright test exercises the main behaviour
2. Screenshots from the test run are shared with the user
3. User explicitly confirms everything is working as expected

For complete file templates, see module-template.md.
