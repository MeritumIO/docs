+++
bookCollapseSection = true
weight = 6
title = "meritum/structured-logging"
description = "Context enrichment, exception translation pipeline, and structured error reporting over PSR-3"
+++

# meritum/structured-logging

`meritum/structured-logging` adds three capabilities on top of a PSR-3 logger: automatic context enrichment on every log call, a typed exception model with structured data, and an exception translation and reporting pipeline. It is designed to be used alongside `meritum/logger` but works with any PSR-3 implementation.

```
composer require meritum/structured-logging
```

## Module Registration

`StructuredLoggingModule` must be registered after whichever module provides `LoggerInterface::class`, because it decorates the logger:

```php
use Meritum\Logger\LoggerModule;
use Meritum\StructuredLogging\StructuredLoggingModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new LoggerModule(),
            new StructuredLoggingModule(),
            // ...
        ];
    }
}
```

The module:
- Registers `CorrelationId` as a shared service
- Registers `CorrelationIdEnricher` (tagged `'log.context.enrichers'`)
- Registers `ExceptionTranslator` (collects `'exception.translator.handlers'` tagged services)
- Registers `ExceptionReporter`
- **Decorates** `LoggerInterface::class` — wraps the existing logger with `ContextEnrichingLogger`

After registration, every log call made through `LoggerInterface` automatically passes through the enrichment pipeline.

## Correlation ID

A `CorrelationId` is generated automatically on every kernel boot — a UUID v4 that is attached to every log entry under the `correlation_id` key.

To propagate an incoming correlation ID from a request header, inject `CorrelationId` and call `set()`:

```php
use Meritum\StructuredLogging\CorrelationId;

final class CorrelationIdMiddleware implements MiddlewareInterface
{
    public function __construct(private readonly CorrelationId $correlationId) {}

    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $incoming = $request->getHeaderLine('X-Correlation-ID');

        if ('' !== $incoming) {
            $this->correlationId->set($incoming);
        }

        return $handler->handle($request);
    }
}
```

`set()` silently ignores the value if it is not a valid UUID v4. The correlation ID is also `Stringable`, so you can include it in response headers:

```php
return $response->withHeader('X-Correlation-ID', (string) $this->correlationId);
```

## Custom Context Enrichers

Implement `ContextEnricher` to append additional fields to every log entry:

```php
use Meritum\StructuredLogging\ContextEnricher;

final class TenantEnricher implements ContextEnricher
{
    public function __construct(private readonly TenantResolver $tenants) {}

    public function enrich(array $context): array
    {
        return $context + ['tenant_id' => $this->tenants->current()->id];
    }
}
```

Register and tag it:

```php
$kernel->define(TenantEnricher::class, fn($c) => new TenantEnricher(
    $c->get(TenantResolver::class)
))->tag('log.context.enrichers');
```

All services tagged `'log.context.enrichers'` are collected at boot and run in sequence on every log call. The `+` union operator means a call-site key takes precedence over an enricher key of the same name.

## Domain Exceptions

Extend `DomainException` to model typed exceptions with structured data for the reporting pipeline:

```php
use Meritum\StructuredLogging\Severity;
use Meritum\StructuredLogging\Exception\DomainException;

final class PaymentFailedException extends DomainException
{
    public function __construct(string $reason, string $transactionId, ?Throwable $previous = null)
    {
        parent::__construct(
            message:   "Payment failed: {$reason}",
            severity:  Severity::Error,
            context:   ['transaction_id' => $transactionId, 'reason' => $reason],
            retryable: true,
            previous:  $previous
        );
    }

    public function getErrorCode(): string
    {
        return 'PAY_0001';
    }
}
```

`DomainException` exposes a `$structuredData` virtual property used by the reporter:

```php
[
    'error_code'  => 'PAY_0001',
    'message'     => 'Payment failed: insufficient funds',
    'severity'    => 'error',
    'retryable'   => true,
    'occurred_at' => '2026-06-17T10:00:00+00:00',
    'detail'      => ['transaction_id' => 'txn_abc', 'reason' => 'insufficient funds'],
    'metadata'    => ['file' => '...', 'line' => 42, 'class' => 'PaymentFailedException'],
]
```

`DomainException` instances passed directly to `ExceptionReporter::report()` are logged as-is — translation is skipped.

## Exception Translation

The translation pipeline converts arbitrary `Throwable` instances into `DomainException`. Implement `TranslationHandler` and tag it:

```php
use Throwable;
use Meritum\StructuredLogging\Severity;
use Meritum\StructuredLogging\TranslationHandler;
use Meritum\StructuredLogging\Exception\DomainException;

final class PdoExceptionHandler implements TranslationHandler
{
    public function matches(Throwable $exception): bool
    {
        return $exception instanceof \PDOException;
    }

    public function handle(Throwable $exception): DomainException
    {
        return new DatabaseException(
            message:  $exception->getMessage(),
            severity: Severity::Critical,
            previous: $exception
        );
    }

    public function priority(): int
    {
        return 100;
    }
}
```

Register and tag it:

```php
$kernel->define(PdoExceptionHandler::class, fn() => new PdoExceptionHandler())
       ->tag('exception.translator.handlers');
```

Handlers are sorted by `priority()` descending — the highest-priority matching handler wins. If no handler matches, the exception is wrapped in `UnknownException` with error code `UNKNOWN_0000`.

## Exception Reporting

Inject `ExceptionReporter` and call `report()` wherever you need to log an exception:

```php
use Meritum\StructuredLogging\ExceptionReporter;

final class ExceptionHandler implements ExceptionHandlerInterface
{
    public function __construct(private readonly ExceptionReporter $reporter) {}

    public function handle(Throwable $e, ServerRequestInterface $request): ResponseInterface
    {
        $domain = $this->reporter->report($e);

        return new JsonResponse(
            ['error' => $domain->getErrorCode()],
            $domain->severity === Severity::Error ? 500 : 400
        );
    }
}
```

`report()` translates the exception, logs it at the appropriate severity using the logger's structured data, and returns the `DomainException` so the caller can use it to build a response.

> `meritum/http-exception-handler` provides a ready-made `ExceptionHandlerInterface` implementation that wires `ExceptionReporter` into the HTTP kernel. See the [http-exception-handler guide](/docs/packages/http-exception-handler/) for details.
