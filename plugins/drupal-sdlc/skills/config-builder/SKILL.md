---
name: config-builder
description: "Generate Drupal config YAML files from plain English descriptions. Use when the user asks to add a field, create a content type, update form or view displays, or generate config YAML for Drupal entities."
argument-hint: "<description of field or config to generate>"
---

# Config Builder

Generate Drupal configuration YAML files from natural language requirements.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml found — ask user for entity types and drush prefix"`
```

## Workflow

1. **Parse the requirement** — extract: field name, field type, entity type (default: node), bundle, cardinality (default: 1), required (default: false), and any type-specific settings.

2. **Generate field storage** — create `config/sync/field.storage.{entity_type}.{field_name}.yml`. Do NOT include `uuid` or `_core` keys. See [config-reference.md](config-reference.md) for the exact structure per field type.

3. **Generate field instance** — create `config/sync/field.field.{entity_type}.{bundle}.{field_name}.yml`. Reference the storage in `dependencies.config`. Set `label`, `required`, `default_value`, and type-specific `settings`.

4. **Update form display** — read the existing `config/sync/core.entity_form_display.{entity_type}.{bundle}.default.yml` and add the new field to the `content` map with the correct widget type and weight.

5. **Update view display** — read the existing `config/sync/core.entity_view_display.{entity_type}.{bundle}.default.yml` and add the new field to the `content` map with the correct formatter type.

6. **Validate output**:
   - No `uuid` keys (Drupal assigns on import)
   - No `_core` keys (only on core-provided config)
   - Field name starts with `field_` and uses snake_case
   - `dependencies` reference correct modules and config
   - `id` follows the pattern `{entity_type}.{field_name}` (storage) or `{entity_type}.{bundle}.{field_name}` (instance)

7. **Config import and export** — after generating config files, read the drush prefix from drupal-claude.yml and run:
```bash
   {drush_prefix} config:import -y
   {drush_prefix} config:export -y
   {drush_prefix} config:status
```
   - `config:import` applies the new config to the database
   - `config:export` immediately re-exports with authoritative UUIDs from the DB
   - `config:status` must show "No differences" before proceeding
   - Always commit the exported files — not the hand-written originals
   - This is mandatory — never skip the export step

8. **Remind the user** to verify at the relevant admin URL after import:
   - New field: `/admin/structure/types/manage/{bundle}/fields`
   - New content type: `/admin/structure/types`
   - New taxonomy: `/admin/structure/taxonomy`

User's request: $ARGUMENTS

## Widget and Formatter Mapping

| Field Type | Widget | Formatter |
|---|---|---|
| string | string_textfield | string |
| text_long | text_textarea | text_default |
| datetime | datetime_default | datetime_default |
| entity_reference | entity_reference_autocomplete | entity_reference_label |
| entity_reference (select) | options_select | entity_reference_label |
| entity_reference_revisions | paragraphs | entity_reference_revisions_entity_view |
| boolean | boolean_checkbox | boolean |
| list_string | options_select | list_default |
| image | image_image | image |
| link | link_default | link |
| file | file_generic | file_default |

## Module Dependencies by Field Type

| Field Type | Module | Storage Module Value |
|---|---|---|
| string | (none — core) | core |
| text_long | text | text |
| datetime | datetime | datetime |
| entity_reference | (none — core) | core |
| entity_reference_revisions | entity_reference_revisions, paragraphs | entity_reference_revisions |
| boolean | (none — core) | core |
| list_string | options | options |
| image | image, file | image |
| link | link | link |
| file | file | file |

## Paragraph Fields

When generating config for paragraph fields:
- Entity type is `paragraph`, not `node`
- Use `entity_reference_revisions` field type (not `entity_reference`)
- Module dependency is `entity_reference_revisions` and `paragraphs`
- Widget is `paragraphs`, formatter is `entity_reference_revisions_entity_view`
- Check `entity_types.paragraph_types` in the project context above for available paragraph types

## Config Splits Awareness

Check `config_splits` in the project context above. If the project uses config splits:
- All standard config goes in `config/sync/`
- If a config needs to behave differently per environment, mention to the user that it may need a config split override in `config/split/{environment}/`
- Never modify files in `config/split/` directories without explicit user approval

## Important Rules

- **NEVER write UUID values** under any circumstances — not even as placeholders like `a1b2c3d4-...`. Leave the `uuid` key absent entirely from all hand-written config YAML. Drupal assigns the correct UUID automatically after `config:import`. The `config:export` step will add it to the file.
- **NEVER write `_core` keys** — only present on core-provided configs, never on custom ones
- **Always run config:export after config:import** — the exported files are the source of truth
- **Never commit hand-written config YAML directly** — always commit the post-export version
- **config:status must be clean before committing** — if it shows differences, resolve them first
- **Never modify `settings.php` or `settings.ddev.php`** without explicit user approval

## Additional Resources

For complete YAML examples of every field type, see [config-reference.md](config-reference.md).
