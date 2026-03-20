---
name: spec-writer
description: "Turn a plain English feature idea into a structured Drupal spec with numbered acceptance criteria, test mapping, and specific Drupal constructs. Always use this first before building anything new."
argument-hint: "<plain English description of what you want to build>"
---

# Spec Writer

Turn a plain English idea into a structured, buildable Drupal spec.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml found"`
```

## Existing Content Types
```
!`ddev drush entity:bundles --entity-type=node 2>/dev/null || echo "Run ddev start first"`
```

## Existing Modules
```
!`ddev drush pm:list --type=module --status=enabled --no-core 2>/dev/null | head -30`
```

## Existing Views
```
!`ddev drush views:list 2>/dev/null | head -20 || echo "No views found"`
```

## User's Idea
$ARGUMENTS

## Instructions

You are helping someone build a Drupal feature.
Translate their plain English idea into a complete, buildable spec.

Ask clarifying questions if anything is ambiguous before writing the spec.

The spec MUST name specific Drupal constructs:
- Content type machine names (e.g. `blog_post`, `event`)
- Field machine names and types (e.g. `field_author` entity_reference to user)
- View machine names and display modes
- Block plugin IDs or block placement
- Template file names (e.g. `node--blog-post.html.twig`)
- Permission strings (e.g. `access content`, `create blog_post content`)
- Menu link placements

## Output Format

Produce the spec in this exact structure:

---

### Feature: {name}

**Intent:**
One sentence — what this does and why.

**What Already Exists:**
List relevant existing content types, fields, views, modules from the project context above.

**What Needs to Be Built:**
List everything new that this feature requires.

**Drupal Implementation Plan:**
- Content types needed (new or existing, with machine names)
- Fields needed (machine name, type, cardinality, required)
- Views needed (machine name, display type, filters, sorts)
- Blocks needed (plugin ID or placed block)
- Templates needed (file name, what it renders)
- Permissions needed (string, what it controls)
- Custom module needed? (yes/no, why)
- Config only or config + code?
- Any contrib modules needed?

**Acceptance Criteria:**
Numbered list with test mapping:

| AC | Criterion | Test File | Test Name |
|---|---|---|---|
| AC-1 | Given X, when Y, then Z | tests/playwright/tests/{category}/{name}.spec.ts | should do X @smoke @fast @readOnly |
| AC-2 | ... | ... | ... |

**Files to Create/Modify:**
| File | Action | Reason |
|---|---|---|
| config/sync/... | Create | new content type |
| web/modules/custom/... | Create | custom logic |

**Test Plan:**
- Playwright: what pages to visit, what to verify, which roles to test as
- PHPUnit: what service logic to unit test (if any)

**Risk Level:** Low / Medium / High
**Reason:** why this risk level

**Open Questions:**
- Anything that needs clarification before building

---

**STOP.** After outputting the spec, ask:
"Spec is ready for your review. Please say 'approved' or 'go ahead' to proceed, or let me know what changes you'd like."

Do NOT proceed to implementation until the user explicitly approves.
