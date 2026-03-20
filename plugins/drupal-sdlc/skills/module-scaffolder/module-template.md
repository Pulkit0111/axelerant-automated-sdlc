# Module Templates

Replace `{module}`, `{Module}`, `{HumanName}`, `{Api}`, `{Description}` with actual values.

## {module}.info.yml

```yaml
name: '{HumanName}'
type: module
description: '{Description}'
package: Custom
core_version_requirement: ^10 || ^11
dependencies: []
```

## {module}.services.yml

```yaml
services:
  {module}.api:
    class: '\Drupal\{module}\{Api}Client'
    arguments:
      - '@config.factory'
      - '@http_client'
      - '@logger.factory'

  {module}.service:
    class: '\Drupal\{module}\{Module}Service'
    arguments:
      - '@entity_type.manager'
      - '@{module}.api'
      - '@logger.factory'
```

### HTTP Client Note

Two patterns exist for HTTP clients:

1. **`@http_client`** (GuzzleHttp\Client) — **default for new modules.**
2. **`@http_client_factory`** (Drupal\Core\Http\ClientFactory) — use when you need custom options per-request via `$this->httpClientFactory->fromOptions([...])->request(...)`.

## {module}.routing.yml

```yaml
{module}.admin_config_main:
  path: '/admin/config/services/{module}'
  defaults:
    _controller: '\Drupal\system\Controller\SystemController::systemAdminMenuBlockPage'
    _title: '{HumanName}'
  requirements:
    _permission: 'access administration pages'

{module}.admin_config_form:
  path: '/admin/config/services/{module}/settings'
  defaults:
    _form: '\Drupal\{module}\Form\SettingsForm'
    _title: '{HumanName} Settings'
  requirements:
    _permission: 'administer site configuration'
```

## {module}.links.menu.yml

```yaml
{module}.admin_config_main:
  title: '{HumanName}'
  description: 'Configure {HumanName}'
  route_name: {module}.admin_config_main
  parent: system.admin_config_services

{module}.admin_config_form:
  title: 'Settings'
  description: 'Configure {HumanName} settings'
  route_name: {module}.admin_config_form
  parent: {module}.admin_config_main
  weight: 0
```

## src/{Api}Client.php

```php
<?php

namespace Drupal\{module};

use GuzzleHttp\Client;
use Drupal\Component\Serialization\Json;
use Drupal\Core\Config\ConfigFactoryInterface;
use Drupal\Core\Logger\LoggerChannelFactoryInterface;
use Drupal\Core\Utility\Error;

/**
 * Provides a service class for {Api} APIs.
 *
 * @see {API_DOCS_URL} API documentation
 */
class {Api}Client {

  /**
   * The API base URL.
   *
   * @see {API_DOCS_URL}
   */
  const API_BASE_URL = 'https://api.example.com/v1';

  /**
   * The immutable configuration object.
   *
   * @var \Drupal\Core\Config\ImmutableConfig
   */
  protected $config;

  /**
   * The HTTP client.
   *
   * @var \GuzzleHttp\Client
   */
  protected $httpClient;

  /**
   * The logger channel.
   *
   * @var \Psr\Log\LoggerInterface
   */
  protected $logger;

  /**
   * {@inheritdoc}
   */
  public function __construct(
    ConfigFactoryInterface $config_factory,
    Client $http_client,
    LoggerChannelFactoryInterface $logger_factory,
  ) {
    $this->config = $config_factory->get('{module}.settings');
    $this->httpClient = $http_client;
    $this->logger = $logger_factory->get('{module}');
  }

  /**
   * Build the request header.
   *
   * @return string[]
   *   The request headers array.
   *
   * @see {API_DOCS_URL} authentication reference
   */
  protected function getHeader(): array {
    $api_token = $this->config->get('api_token');
    if (empty($api_token)) {
      throw new \Exception('API token is required.');
    }

    return [
      'Authorization' => "Bearer {$api_token}",
      'Accept' => 'application/json',
    ];
  }

  /**
   * Helper function to make a request.
   *
   * @param string $http_verb
   *   The HTTP verb (GET, POST, PUT, DELETE).
   * @param string $endpoint
   *   The API endpoint path.
   * @param array $options
   *   Additional request options.
   *
   * @return array|null
   *   Decoded JSON response or NULL for 204 responses.
   *
   * @see {API_DOCS_URL}
   */
  protected function makeRequest(string $http_verb, string $endpoint, array $options = []): ?array {
    $url = self::API_BASE_URL . $endpoint;
    $params = ['headers' => $this->getHeader()] + $options;

    try {
      $response = $this->httpClient->request($http_verb, $url, $params);
      $statusCode = $response->getStatusCode();

      if ($statusCode === 204) {
        return NULL;
      }

      return Json::decode($response->getBody()->getContents());
    }
    catch (\Exception $e) {
      Error::logException($this->logger, $e);
      throw $e;
    }
  }
}
```

## src/Form/SettingsForm.php

```php
<?php

namespace Drupal\{module}\Form;

use Drupal\Core\Form\ConfigFormBase;
use Drupal\Core\Form\FormStateInterface;

/**
 * Provides a configuration form for {HumanName}.
 */
class SettingsForm extends ConfigFormBase {

  /**
   * {@inheritdoc}
   */
  public function getFormId(): string {
    return '{module}_settings_form';
  }

  /**
   * {@inheritdoc}
   */
  protected function getEditableConfigNames(): array {
    return ['{module}.settings'];
  }

  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state): array {
    $config = $this->config('{module}.settings');

    $form['api_token'] = [
      '#type' => 'textfield',
      '#title' => $this->t('API Token'),
      '#required' => TRUE,
      '#default_value' => $config->get('api_token'),
    ];

    return parent::buildForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state): void {
    $this->config('{module}.settings')
      ->set('api_token', $form_state->getValue('api_token'))
      ->save();

    parent::submitForm($form, $form_state);
  }

}
```

## config/schema/{module}.schema.yml

```yaml
{module}.settings:
  type: config_object
  label: '{HumanName} Configuration'
  mapping:
    api_token:
      type: text
      label: 'API Token'
```

## config/install/{module}.settings.yml

```yaml
api_token: ''
```

## {module}.module (if hooks are needed)

```php
<?php

/**
 * @file
 * Hook implementations for {HumanName}.
 */

use Drupal\Core\Entity\EntityInterface;

/**
 * Implements hook_user_login().
 */
function {module}_user_login($account) {
  /** @var \Drupal\{module}\{Module}Service $service */
  $service = \Drupal::service('{module}.service');
  // Add integration logic here.
}
```

## drush.services.yml

Separate from the main `{module}.services.yml`:

```yaml
services:
  {module}.commands:
    class: '\Drupal\{module}\Drush\Commands\{Module}Commands'
    arguments:
      - '@{module}.api'
      - '@logger.factory'
    tags:
      - { name: drush.command }
```

## src/Drush/Commands/{Module}Commands.php

```php
<?php

namespace Drupal\{module}\Drush\Commands;

use Drupal\{module}\{Api}Client;
use Drupal\Core\Logger\LoggerChannelFactoryInterface;
use Drush\Commands\DrushCommands;

/**
 * Drush commands for {HumanName}.
 *
 * @see {API_DOCS_URL}
 */
class {Module}Commands extends DrushCommands {

  /**
   * The API client.
   *
   * @var \Drupal\{module}\{Api}Client
   */
  protected $apiClient;

  /**
   * {@inheritdoc}
   */
  public function __construct(
    {Api}Client $api_client,
    LoggerChannelFactoryInterface $logger_factory,
  ) {
    parent::__construct();
    $this->apiClient = $api_client;
    $this->logger = $logger_factory->get('{module}');
  }

  /**
   * Syncs data from {Api}.
   *
   * @command {module}:sync
   * @aliases {abbr}-sync
   * @usage drush {module}:sync
   *   Syncs all data from the {Api} API.
   *
   * @see {API_DOCS_URL}
   */
  public function sync(): void {
    $this->logger()->success(dt('Starting {Api} sync...'));
    // Implementation here.
    $this->logger()->success(dt('{Api} sync complete.'));
  }

}
```

## src/Batch/{Operation}BatchManager.php

```php
<?php

namespace Drupal\{module}\Batch;

use Drupal\Core\StringTranslation\StringTranslationTrait;

/**
 * Manages batch processing for {Operation}.
 *
 * @see {API_DOCS_URL}
 */
class {Operation}BatchManager {

  use StringTranslationTrait;

  /**
   * Creates the batch definition.
   *
   * @param array $items
   *   The items to process.
   *
   * @return array
   *   The batch definition array.
   */
  public static function createBatch(array $items): array {
    $operations = [];
    foreach ($items as $item) {
      $operations[] = [[self::class, 'processItem'], [$item]];
    }

    return [
      'title' => t('Processing...'),
      'operations' => $operations,
      'finished' => [self::class, 'finished'],
    ];
  }

  /**
   * Processes a single batch item.
   *
   * @param mixed $item
   *   The item to process.
   * @param array $context
   *   The batch context.
   */
  public static function processItem($item, array &$context): void {
    /** @var \Drupal\{module}\{Module}Service $service */
    $service = \Drupal::service('{module}.service');
    $context['results'][] = $item;
    $context['message'] = t('Processing item...');
  }

  /**
   * Batch finished callback.
   *
   * @param bool $success
   *   Whether the batch completed successfully.
   * @param array $results
   *   The results from processing.
   * @param array $operations
   *   Any remaining operations.
   */
  public static function finished(bool $success, array $results, array $operations): void {
    if ($success) {
      \Drupal::messenger()->addStatus(t('Processed @count items.', [
        '@count' => count($results),
      ]));
    }
    else {
      \Drupal::messenger()->addError(t('An error occurred during processing.'));
    }
  }

}
```

## Event Subscriber Template (optional)

### src/EventSubscriber/{Event}Subscriber.php

```php
<?php

namespace Drupal\{module}\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Drupal\Core\Logger\LoggerChannelFactoryInterface;

/**
 * Subscribes to {event} events.
 */
class {Event}Subscriber implements EventSubscriberInterface {

  /**
   * The logger channel.
   *
   * @var \Psr\Log\LoggerInterface
   */
  protected $logger;

  /**
   * {@inheritdoc}
   */
  public function __construct(LoggerChannelFactoryInterface $logger_factory) {
    $this->logger = $logger_factory->get('{module}');
  }

  /**
   * {@inheritdoc}
   */
  public static function getSubscribedEvents(): array {
    return [];
  }

}
```

### Services entry for event subscriber

```yaml
  {module}.event_subscriber:
    class: '\Drupal\{module}\EventSubscriber\{Event}Subscriber'
    arguments:
      - '@logger.factory'
    tags:
      - { name: 'event_subscriber' }
```
