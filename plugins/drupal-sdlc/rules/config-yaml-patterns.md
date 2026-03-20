---
paths:
  - "config/**/*.yml"
  - "config/sync/**/*.yml"
  - "config/split/**/*.yml"
---

# Drupal Config YAML Patterns

## File Naming

- Field storage: `field.storage.{entity_type}.{field_name}.yml`
- Field instance: `field.field.{entity_type}.{bundle}.{field_name}.yml`
- Form display: `core.entity_form_display.{entity_type}.{bundle}.{mode}.yml`
- View display: `core.entity_view_display.{entity_type}.{bundle}.{mode}.yml`

## Project Entity Types

Read `drupal-claude.yml` -> `entity_types` for this project's content types, taxonomies, and paragraph types.

## Critical Rules

- Never generate UUIDs — Drupal assigns them on `drush config:import`.
- Never add `_core` key — only present on core-provided configs.
- Never set `locked: true` unless explicitly requested.
- Always set `langcode: en`, `status: true`.
- Field names must start with `field_` and use snake_case.
- `cardinality: 1` for single-value, `-1` for unlimited.

## Dependencies

- `field.storage.*.yml` lists module dependencies under `dependencies.module`.
- `field.field.*.yml` references its storage and bundle under `dependencies.config`.

## Validation

- CI validates via `drush config:import`. Test locally with the project's drush prefix (see `drupal-claude.yml`).
- After generating config, always remind the user to import and verify.
