+++
title = "DomainException"
description = "Abstract base class for typed domain exceptions with structured data"
weight = 2
+++

# DomainException

`Meritum\StructuredLogging\Exception\DomainException`

Abstract base class for typed domain exceptions. Extends `\Exception`. Provides structured data for the reporting pipeline.

## Constructor

```php
public function __construct(
    string $message,
    Severity $severity,
    array $context = [],
    bool $retryable = false,
    int $code = 0,
    ?Throwable $previous = null
)
```

| Parameter | Description |
|---|---|
| `$message` | Human-readable error message |
| `$severity` | [`Severity`](/docs/packages/structured-logging/api/severity/) enum value |
| `$context` | Domain-specific detail as `array<string, mixed>` |
| `$retryable` | Whether the operation can be retried |
| `$code` | Exception code (passed to `\Exception`) |
| `$previous` | Previous throwable for chaining |

## Abstract method

```php
abstract public function getErrorCode(): string;
```

Returns a namespaced error code in the format `PREFIX_NNNN` (e.g. `PAY_0001`, `DB_0003`). Used in the structured data and by callers to identify the failure type.

## Public properties

```php
public protected(set) Severity $severity;
public protected(set) array $context;
public protected(set) bool $retryable;
public protected(set) DateTimeImmutable $occurredAt;
```

These are set in the constructor and are readable from outside the class but can only be written by the class itself and its subclasses.

## Virtual properties

```php
public array $structuredData { get }
```

Returns the full structured representation used by `ExceptionReporter`:

```php
[
    'error_code'  => $this->getErrorCode(),
    'message'     => $this->getMessage(),
    'severity'    => $this->severity->value,
    'retryable'   => $this->retryable,
    'occurred_at' => '2026-06-17T10:00:00+00:00',
    'detail'      => $this->context,
    'metadata'    => ['file' => '...', 'line' => 42, 'class' => '...'],
]
```

```php
public array $metadata { get }
```

Lazy-loaded. Contains `file`, `line`, and `class` of the exception. Populated on first access.

## Example

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
