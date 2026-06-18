+++
title = "ExceptionTranslator"
description = "Converts any Throwable into a DomainException via the translation handler pipeline"
weight = 4
+++

# ExceptionTranslator

`Meritum\StructuredLogging\ExceptionTranslator`

Converts a `Throwable` into a [`DomainException`](/docs/packages/structured-logging/api/domain-exception/) by running it through the registered [`TranslationHandler`](/docs/packages/structured-logging/api/translation-handler/) pipeline. Registered by `StructuredLoggingModule` as `ExceptionTranslator::class`.

## Methods

```php
public function translate(Throwable $exception): DomainException;
```

- If `$exception` is already a `DomainException`, it is returned as-is.
- Otherwise, handlers are checked in priority order. The first matching handler's `handle()` result is returned.
- If no handler matches, the exception is wrapped in `UnknownException`.

## Example

The translator is used internally by `ExceptionReporter`. Inject it directly only if you need to translate without logging:

```php
use Meritum\StructuredLogging\ExceptionTranslator;

final class SomeService
{
    public function __construct(private readonly ExceptionTranslator $translator) {}

    public function execute(): void
    {
        try {
            $this->doWork();
        } catch (\Throwable $e) {
            $domain = $this->translator->translate($e);
            // inspect $domain->severity, $domain->retryable, etc.
        }
    }
}
```
