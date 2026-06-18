+++
title = "CorrelationId"
description = "Shared UUID that is attached to every log entry"
weight = 6
+++

# CorrelationId

`Meritum\StructuredLogging\CorrelationId`

A shared service holding a UUID v4 that is automatically attached to every log entry via `CorrelationIdEnricher`. Registered as a singleton by `StructuredLoggingModule`.

A new UUID is generated on each kernel boot. Inject `CorrelationId` and call `set()` to propagate an incoming correlation ID from a request header.

## Properties

```php
public private(set) string $uuid;
```

The current UUID. Readable by anyone; writable only by the class itself. Cast the object to string to get the value — `CorrelationId` implements `Stringable`.

## Methods

```php
public function set(string $uuid): void;
```

Updates the current UUID. Silently ignores the value if it is not a valid UUID v4.

## Example

Propagate a correlation ID from an incoming request header:

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

        $response = $handler->handle($request);

        return $response->withHeader('X-Correlation-ID', (string) $this->correlationId);
    }
}
```
