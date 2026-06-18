+++
title = "ContextEnricher"
description = "Appends fields to the log context on every log call"
weight = 1
+++

# ContextEnricher

`Meritum\StructuredLogging\ContextEnricher`

Appends fields to the log context before it is passed to the underlying PSR-3 logger. Register implementations tagged with `'log.context.enrichers'` to include them in the pipeline.

## Methods

```php
/** @param mixed[] $context @return mixed[] */
public function enrich(array $context): array;
```

Receives the context array from the log call and returns an augmented copy. Use the `+` union operator to avoid overwriting keys the caller set explicitly:

```php
return $context + ['tenant_id' => $this->resolver->current()->id];
```

## Built-in enricher

`CorrelationIdEnricher` is registered automatically by `StructuredLoggingModule`. It adds `'correlation_id'` to every log entry using the shared `CorrelationId` service.

## Example

```php
use Meritum\StructuredLogging\ContextEnricher;

final class AppVersionEnricher implements ContextEnricher
{
    public function enrich(array $context): array
    {
        return $context + ['app_version' => '1.4.2'];
    }
}
```

```php
$kernel->define(AppVersionEnricher::class, fn() => new AppVersionEnricher())
       ->tag('log.context.enrichers');
```
