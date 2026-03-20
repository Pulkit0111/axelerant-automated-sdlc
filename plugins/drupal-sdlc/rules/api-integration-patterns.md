---
paths:
  - "web/modules/custom/**"
---

# API Integration Patterns

## Architecture Flow

API Client Class (HTTP wrapper)
  injected into
Business Logic Service (maps external data to Drupal entities)
  triggered by
Event Subscriber / Hook / Cron / Drush Command
  configured by
Config Form (extends ConfigFormBase, stores API credentials)

## Authentication Patterns

| Auth Type | Header Format | Token Storage |
|---|---|---|
| Bearer Token | Authorization: Bearer {token} | Drupal config or Key module |
| OAuth2 | Authorization: Bearer {token} | drupal/oauth2_client module |
| API Key (header) | X-Api-Key: {key} | Drupal config |
| API Key (query) | ?api_key={key} | Drupal config |

Check `drupal-claude.yml` -> `api_integrations` for this project's specific integrations.

## HTTP Client Usage

Default pattern (@http_client):
  public function __construct(
    ConfigFactoryInterface $config_factory,
    Client $http_client,
    LoggerChannelFactoryInterface $logger_factory,
  ) {
    $this->httpClient = $http_client;
  }

Use @http_client_factory only when per-request options are needed.

## Error Handling Pattern

All API clients must use try/catch with Error::logException:
  try {
    $response = $this->httpClient->request($method, $url, $params);
    return Json::decode($response->getBody()->getContents());
  }
  catch (\Exception $e) {
    Error::logException($this->logger, $e);
    throw $e;
  }

## When Generating New API Integration Code

1. Always ask the user for the API documentation URL
2. Generate the API client with exact endpoints, HTTP methods, and headers from docs
3. Include @see tags in every class and method docblock
4. Use existing API integration modules in the project as reference
5. Include error handling with Error::logException in every HTTP call
6. NEVER use \Drupal::service() in classes — always constructor injection
7. Generate a config form for API credentials
8. Generate a config schema file
