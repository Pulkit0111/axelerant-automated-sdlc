---
name: test-writer
description: "Generate Playwright specs and PHPUnit tests for Drupal projects. Use when the user asks to write tests, generate test specs, improve test coverage, create Playwright tests, or write unit tests for services."
argument-hint: "<what to test: feature, module, or component>"
---

# Test Writer

Generate tests following established patterns.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null | grep -A3 'playwright' || echo "No playwright config"`
```

## Playwright Config
```
!`cat tests/playwright/playwright.config.ts 2>/dev/null | head -30 || echo "No Playwright config found at tests/playwright/playwright.config.ts"`
```

## Existing Test Structure
```
!`find tests/playwright/tests -type f -name '*.spec.ts' 2>/dev/null | head -20 || echo "No Playwright tests found"`
```

## User's Request

$ARGUMENTS

## Playwright Tests

### File Placement

Place specs under `tests/playwright/tests/{category}/{name}.spec.ts`:
- `smoke/` — fast, read-only checks for basic site functionality
- `content/{type}/` — content type-specific tests
- `authoring/` — content creation and editing
- `forms/` — form interactions and validation
- `accessibility/` — keyboard nav, skip links, ARIA
- `permissions/` — role-based access
- `integrations/` — external API integration tests
- `regression/` — bug regression tests

### Login Helper — Always Use This

Never duplicate login code in test files.
Always import and use the shared helper:
```typescript
import { loginAsAdmin } from '../helpers/auth';

test.beforeEach(async ({ page }) => {
  await loginAsAdmin(page);
});
```

The helper is at `tests/playwright/helpers/auth.ts`.
Never write manual login steps (goto /user/login, fill username/password, click Log in) in any test file — always use the helper instead.

### Multi-Role Testing

For features with role-based access, scaffold tests for multiple roles:

```typescript
import { loginAsAdmin, loginAsUser, logout } from '../helpers/auth';

test.describe('Feature as anonymous', () => {
  test('anonymous user sees public content @permissions @fast @readOnly', async ({ page }) => {
    await page.goto('/content-page');
    await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
  });
});

test.describe('Feature as authenticated', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsUser(page, 'authenticated_user', 'password');
  });

  test('authenticated user can access feature @permissions @fast @readOnly', async ({ page }) => {
    await page.goto('/content-page');
    // assertions for authenticated role
  });
});

test.describe('Feature as admin', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsAdmin(page);
  });

  test('admin can manage feature @permissions @fast @readOnly', async ({ page }) => {
    await page.goto('/admin/content');
    // assertions for admin role
  });
});
```

If the project only has `loginAsAdmin()`, generate a `loginAsUser()` helper alongside:
```typescript
export async function loginAsUser(page: Page, username: string, password: string) {
  await page.goto('/user/login');
  await page.getByRole('textbox', { name: /Username/i }).fill(username);
  await page.getByRole('textbox', { name: /Password/i }).fill(password);
  await page.getByRole('button', { name: /Log in/i }).click();
  await page.waitForURL('**/user/**');
}
```

### Mandatory Tags (in test title)

Every test title must end with tags:
- **Cost:** `@fast` (< 5s), `@medium` (5-15s), `@slow` (> 15s)
- **Intent:** `@smoke`, `@regression`, `@visual`, `@accessibility`
- **Behavior:** `@readOnly` (no data changes) or `@mutates` (creates/modifies content)

### Drupal UI and node forms

- **Title:** Use `getByRole('textbox', { name: /Title \*/i })` to target the required node title only
- **Body (CKEditor5):** Use `page.getByRole('textbox', { name: 'Rich Text Editor' })` — never use `#edit-body-0-value` as CKEditor5 hides the textarea
- **Submit (Gin admin theme):** Use `page.getByRole('button', { name: 'Save' })` — never use `#edit-submit` as it matches multiple elements on the page
- **Featured Image:** Use `page.getByRole('button', { name: 'Featured Image' })` — it renders as a collapsible fieldset button
- **Select fields:** Use `page.getByRole('combobox', { name: 'Field Label' })`
- **Autocomplete fields:** Use `page.getByRole('textbox', { name: 'Field Label' })`

### Selector Priority

Always prefer in this order:
1. `getByRole` — most reliable, matches accessible names
2. `getByLabel` — for form fields with explicit labels
3. `getByPlaceholder` — for inputs with placeholder text
4. `getByText` — for visible text content
5. `locator('[data-drupal-selector="..."]')` — only as last resort

Never use:
- `#edit-submit` — matches multiple elements
- `#edit-body-0-value` — hidden by CKEditor5
- Raw CSS class selectors — fragile, change with theme updates

### Rules

- Use relative URLs only — `page.goto('/dashboard')` not full URLs
- Use regex with case-insensitive flag: `/Text/i`
- No random data — use fixed, deterministic values
- Always use `await` for every Playwright action
- Always assert something after every user action

---

## PHPUnit Tests

### File Placement

`web/modules/custom/{module}/tests/src/Unit/{Class}Test.php`

### Namespace

`Drupal\Tests\{module}\Unit`

### Key PHPUnit Rules

1. Always extend `Drupal\Tests\UnitTestCase`
2. Always include `@group {module}` for test filtering
3. Unit tests must not require a database connection — mock everything
4. Use `GuzzleHttp\Psr7\Response` for HTTP response mocking
5. Use `$this->createMock()` for all service dependencies

## Additional Resources

For complete real-world test examples, see [test-examples.md](test-examples.md).
