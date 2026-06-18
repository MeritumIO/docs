+++
bookCollapseSection = true
weight = 7
title = "meritum/http-exception-handler"
description = "Translates exceptions into structured JSON error responses using the meritum/structured-logging pipeline"
+++

# meritum/http-exception-handler

`meritum/http-exception-handler` wires `meritum/structured-logging` into the HTTP kernel's exception handling contract. It reports every unhandled exception through the structured logging pipeline and returns a consistent JSON error response.

```
composer require meritum/http-exception-handler
```

## Dependencies

This package requires both `meritum/http` and `meritum/structured-logging` to be present and their modules registered before `ExceptionHandlerModule`.

## Module Registration

```php
use Meritum\Logger\LoggerModule;
use Meritum\StructuredLogging\StructuredLoggingModule;
use Meritum\HttpExceptionHandler\ExceptionHandlerModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new LoggerModule(),
            new StructuredLoggingModule(),
            new ExceptionHandlerModule(),
            // ...
        ];
    }
}
```

`ExceptionHandlerModule` registers two things:

- `HttpExceptionTranslationHandler` — tagged `'exception.translator.handlers'`; translates `HttpExceptionInterface` exceptions into `HttpDomainException`
- `ExceptionHandlerInterface::class` — the HTTP kernel picks this up automatically and routes unhandled exceptions through it

No additional wiring is required. The `ExceptionHandler` receives `ExceptionReporter` from the container, which was set up by `StructuredLoggingModule`.

## Error Response Shape

Every unhandled exception produces a JSON response:

```json
{
  "code": "HTTP_404",
  "status": 404,
  "title": "Not Found",
  "detail": "The requested resource was not found"
}
```

For `HttpExceptionInterface` exceptions, the status code and title come from the exception itself. For all other exceptions — those translated by a `TranslationHandler` or wrapped as `UnknownException` — the status is always `500` and the title is `'Unexpected Error'`:

```json
{
  "code": "PAY_0001",
  "status": 500,
  "title": "Unexpected Error",
  "detail": "Payment failed: insufficient funds"
}
```

The `code` field is the `DomainException::getErrorCode()` value from the translated exception.

## Severity Mapping

HTTP exceptions are logged at a severity that reflects whether the condition is a system problem or expected client behaviour:

| Status range | Severity | Rationale |
|---|---|---|
| 4xx | `debug` | Client errors — expected, not a system alert |
| 503 | `debug` | Service unavailable is typically a known state |
| 5xx (other) | `error` | Unexpected server-side failure |

Non-HTTP exceptions wrapped as `UnknownException` are logged at `error` severity.

## Custom Exception Types

Domain exceptions thrown inside request handlers are translated using the same `TranslationHandler` pipeline from `meritum/structured-logging`. Register a handler for each exception type you want to control:

```php
use Throwable;
use Meritum\StructuredLogging\Severity;
use Meritum\StructuredLogging\TranslationHandler;
use Meritum\StructuredLogging\Exception\DomainException;

final class PaymentExceptionHandler implements TranslationHandler
{
    public function matches(Throwable $exception): bool
    {
        return $exception instanceof PaymentException;
    }

    public function handle(Throwable $exception): DomainException
    {
        return new PaymentFailedException(
            $exception->getReason(),
            $exception->getTransactionId(),
            $exception
        );
    }

    public function priority(): int
    {
        return 50;
    }
}
```

```php
$kernel->define(PaymentExceptionHandler::class, fn() => new PaymentExceptionHandler())
       ->tag('exception.translator.handlers');
```

Unrecognised exceptions that have no matching handler fall through to `UnknownException` with error code `UNKNOWN_0000` and a 500 response.
