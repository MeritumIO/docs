+++
title = "ExceptionReporter"
description = "Translates and logs an exception, returning the DomainException"
weight = 5
+++

# ExceptionReporter

`Meritum\StructuredLogging\ExceptionReporter`

Translates a `Throwable` into a [`DomainException`](/docs/packages/structured-logging/api/domain-exception/) and logs it using the structured data. Registered by `StructuredLoggingModule` as `ExceptionReporter::class`.

## Methods

```php
public function report(Throwable $exception): DomainException;
```

1. Passes `$exception` through `ExceptionTranslator::translate()`.
2. Calls `$logger->log($severity, $message, $structuredData)` on the translated exception.
3. Returns the `DomainException` so the caller can use it to construct a response.

## Example

```php
use Meritum\StructuredLogging\Severity;
use Meritum\StructuredLogging\ExceptionReporter;

final class ExceptionHandler implements ExceptionHandlerInterface
{
    public function __construct(private readonly ExceptionReporter $reporter) {}

    public function handle(Throwable $e, ServerRequestInterface $request): ResponseInterface
    {
        $domain = $this->reporter->report($e);

        $status = match ($domain->severity) {
            Severity::Error, Severity::Critical => 500,
            default                             => 400,
        };

        return new JsonResponse(['error' => $domain->getErrorCode()], $status);
    }
}
```
