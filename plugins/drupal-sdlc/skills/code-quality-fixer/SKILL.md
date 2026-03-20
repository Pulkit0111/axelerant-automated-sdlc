---
name: code-quality-fixer
description: "Fix PHPStan, PHPCS, twigcs, and yamllint violations from error output. Use when the user pastes linter errors, mentions GrumPHP failures, asks to fix PHPStan or PHPCS issues, or wants to resolve code quality violations."
argument-hint: "<paste error output or file path>"
---

# Code Quality Fixer

Fix PHPStan, PHPCS, twigcs, and yamllint violations in Drupal custom modules.

## Project Quality Stack

!`cat grumphp.yml.dist 2>/dev/null || cat grumphp.yml 2>/dev/null || echo "No GrumPHP config found"`

## Project Config

!`cat drupal-claude.yml 2>/dev/null | grep -A2 'quality_command' || echo "quality_command: not configured"`

## Errors to fix

$ARGUMENTS

## Workflow

1. Identify the error source (PHPStan, PHPCS, twigcs, or yamllint format)
2. Fix each error following the patterns below
3. Apply fixes file by file
4. Re-run the quality command from drupal-claude.yml to verify fixes

### PHPStan Fixes

| Error Pattern | Fix |
|---|---|
| Missing return type | Add `: void`, `: string`, `: array`, `: int`, `: bool`, or `: mixed` |
| Missing parameter type | Add type hint |
| Cannot access property on null | Add null check: `if ($entity === NULL) { return; }` |
| Null passed to non-nullable | Add null coalescing: `$value ?? ''` |
| Property has no type | Add typed property: `protected string $value;` |

### PHPCS Fixes (Drupal Standard)

| Error Pattern | Fix |
|---|---|
| Missing file docblock | Add `/** @file */` |
| Missing function docblock | Add `/** {@inheritdoc} */` |
| Inline control structure | Convert to braced form |
| Short array syntax | Use `[]` not `array()` |

### DrupalPractice Fixes

| Error Pattern | Fix |
|---|---|
| `\Drupal::service()` in class | Refactor to constructor injection |
| `\Drupal::config()` in class | Inject `@config.factory` |

### Twigcs / yamllint Fixes

- Twig: space after `:` in hash literals, commas between values, newline after block tags
- YAML: 2-space indentation, no trailing spaces, no duplicate keys

## Important Notes

- Only fix files in `web/modules/custom/`. Never modify contrib, core, or vendor.
- After fixing type hints, ensure method signatures match any interface or parent class.
