# Test Examples — Playwright & PHPUnit

## Smoke Test: Navigation

```typescript
import { test, expect } from '@playwright/test';

test.describe('Navigation smoke tests', () => {

  test('main navigation links work correctly @smoke @fast @navigation @readOnly', async ({ page }) => {
    await page.goto('/dashboard');

    // Test Home link
    await page.getByRole('link', { name: /^Home$/i }).click();
    await expect(page).toHaveURL(/\/dashboard/);
  });

  test('breadcrumb navigation works @smoke @fast @navigation @readOnly', async ({ page }) => {
    await page.goto('/admin');

    const breadcrumb = page.getByRole('navigation', { name: /Breadcrumb/i });
    await expect(breadcrumb).toBeVisible();
    await expect(breadcrumb.getByRole('link', { name: /Home/i })).toBeVisible();
  });

  test('skip to main content link works @smoke @fast @accessibility @readOnly', async ({ page }) => {
    await page.goto('/');

    const skipLink = page.getByRole('link', { name: /Skip to main content/i });
    await expect(skipLink).toBeVisible();
    await expect(skipLink).toHaveAttribute('href', /#main-content/);
  });
});
```

## Forms Test: Search Functionality

```typescript
import { test, expect } from '@playwright/test';

test.describe('Search functionality', () => {

  test('search box accepts input @forms @fast @readOnly', async ({ page }) => {
    await page.goto('/admin/content');

    const searchBox = page.getByPlaceholder(/Search/i).first();
    await expect(searchBox).toBeVisible();
    await expect(searchBox).toBeEditable();

    await searchBox.fill('Test Content');
    await expect(searchBox).toHaveValue('Test Content');
  });

  test('checkbox filter is toggleable @forms @fast @readOnly', async ({ page }) => {
    await page.goto('/admin/content');

    const checkbox = page.getByRole('checkbox').first();
    if (await checkbox.isVisible()) {
      await checkbox.check();
      await expect(checkbox).toBeChecked();
      await checkbox.uncheck();
      await expect(checkbox).not.toBeChecked();
    }
  });
});
```

## Multi-Role Permission Test

```typescript
import { test, expect } from '@playwright/test';
import { loginAsAdmin } from '../helpers/auth';

test.describe('Content access by role', () => {

  test('anonymous user cannot access admin @permissions @fast @readOnly', async ({ page }) => {
    const response = await page.goto('/admin');
    expect(response?.status()).toBe(403);
  });

  test('admin can access content overview @permissions @fast @readOnly', async ({ page }) => {
    await loginAsAdmin(page);
    await page.goto('/admin/content');
    await expect(page.getByRole('heading', { name: /Content/i })).toBeVisible();
  });
});
```

## Key Patterns

1. **Imports:** Always `import { test, expect } from '@playwright/test'`
2. **Tags in title:** Every test ends with `@intent @cost @behavior` tags
3. **Selectors:** `getByRole` > `getByPlaceholder` > `getByText` > `locator`
4. **URLs:** Always relative (`'/dashboard'`), never absolute
5. **Assertions:** `toBeVisible()`, `toHaveURL()`, `toHaveValue()`, `toBeChecked()`
6. **Regex:** Case-insensitive with `/pattern/i`
7. **No random data:** fixed strings like `'Test Content'`

## PHPUnit Example: API Service Test

```php
<?php

namespace Drupal\Tests\my_module\Unit;

use Drupal\Tests\UnitTestCase;
use Drupal\my_module\MyApiClient;
use Drupal\Core\Config\ConfigFactoryInterface;
use Drupal\Core\Config\ImmutableConfig;
use Drupal\Core\Logger\LoggerChannelFactoryInterface;
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Response;
use Psr\Log\LoggerInterface;

/**
 * Tests for MyApiClient.
 *
 * @group my_module
 * @coversDefaultClass \Drupal\my_module\MyApiClient
 */
class MyApiClientTest extends UnitTestCase {

  protected MyApiClient $service;
  protected Client $httpClient;

  protected function setUp(): void {
    parent::setUp();

    $config = $this->createMock(ImmutableConfig::class);
    $config->method('get')->with('api_token')->willReturn('test-token');
    $configFactory = $this->createMock(ConfigFactoryInterface::class);
    $configFactory->method('get')->willReturn($config);

    $this->httpClient = $this->createMock(Client::class);

    $logger = $this->createMock(LoggerInterface::class);
    $loggerFactory = $this->createMock(LoggerChannelFactoryInterface::class);
    $loggerFactory->method('get')->willReturn($logger);

    $this->service = new MyApiClient($configFactory, $this->httpClient, $loggerFactory);
  }

  /**
   * Tests error handling logs the exception.
   */
  public function testExceptionIsLoggedOnApiFailure(): void {
    $this->httpClient->method('request')
      ->willThrowException(new \Exception('API unavailable'));

    $this->expectException(\Exception::class);
    // Call a public method that triggers makeRequest.
  }

}
```

### Key PHPUnit Patterns

1. **File location:** `web/modules/custom/{module}/tests/src/Unit/{Class}Test.php`
2. **Namespace:** `Drupal\Tests\{module}\Unit`
3. **Base class:** Always extend `Drupal\Tests\UnitTestCase`
4. **Mock HTTP responses:** Use `GuzzleHttp\Psr7\Response` for mocking API returns
5. **Group tag:** Always include `@group {module}` for test filtering
6. **No database:** Unit tests must not require a database connection — mock everything
