+++
bookCollapseSection = true
weight = 5
title = "meritum/logger"
description = "Minimal PSR-3 logger that writes newline-delimited JSON to stdout"
+++

# meritum/logger

`meritum/logger` is a minimal PSR-3 logger that writes newline-delimited JSON to `stdout`. It is designed for containerised environments where log aggregation is handled externally.

```
composer require meritum/logger
```

## Module Registration

Add `LoggerModule` to your module repository:

```php
use Meritum\Logger\LoggerModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new LoggerModule(),
            // ...
        ];
    }
}
```

The module registers a shared `Psr\Log\LoggerInterface` in the container. Inject it wherever logging is needed:

```php
use Psr\Log\LoggerInterface;

final class CreateUserHandler implements RequestHandlerInterface
{
    public function __construct(private readonly LoggerInterface $logger) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        $this->logger->info('User created', ['user_id' => $user->id]);

        // ...
    }
}
```

## Configuration

The minimum log level is controlled by the `LOG_LEVEL` environment variable. If not set, it defaults to `debug` in `Environment::Development` and `info` in all other environments.

```
LOG_LEVEL=warning
```

Valid values (case-insensitive): `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

Messages below the minimum level are silently discarded. All other configuration is automatic — no additional setup is required.

## Log Entry Format

Each log call writes a single JSON object followed by a newline:

```json
{
  "timestamp": "2026-06-17T10:00:00.000+00:00",
  "level": "info",
  "severity": "info",
  "message": "User created",
  "context": {
    "user_id": 42
  }
}
```

| Field | Description |
|---|---|
| `timestamp` | RFC 3339 with millisecond precision |
| `level` | The PSR-3 level as passed to the log call |
| `severity` | Reduced severity bucket (see below) |
| `message` | The log message |
| `context` | The context array passed to the log call |

## Severity Mapping

PSR-3 defines eight log levels. The logger maps these to five severity buckets in the `severity` field, which is intended for use by structured log sinks and alerting pipelines. The original PSR-3 `level` is always preserved.

| PSR-3 level | `severity` |
|---|---|
| `emergency` | `critical` |
| `alert` | `critical` |
| `critical` | `critical` |
| `error` | `error` |
| `warning` | `warning` |
| `notice` | `info` |
| `info` | `info` |
| `debug` | `debug` |

> `meritum/structured-logging` builds on this logger to add enrichment, channel routing, and pipeline processing. See the [structured-logging guide](/docs/packages/structured-logging/) for details.
