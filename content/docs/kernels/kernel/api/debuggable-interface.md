+++
title = "DebuggableInterface"
description = "Opt-in contract for contributing debug information to the kernel"
weight = 10
+++

# DebuggableInterface

`Georgeff\Kernel\Debug\DebuggableInterface`

An opt-in contract for contributing debug information to the kernel's debug snapshot. Only relevant when `debug: true` is passed to the kernel constructor.

## Methods

```php
public function getDebugInfo(): array;
```

Returns an `array<string, mixed>` of debug information. The structure is up to the implementing class.

## Auto-aggregation

`Kernel::getDebugInfo()` automatically includes the debug output of resolved services that implement `DebuggableInterface`, provided the service registrar implements `ResolvingAwareServiceRegistrar`. After each factory resolution, the kernel checks whether the resolved instance implements this interface and stores it. Those results appear in `getDebugInfo()` under `services.resolved[id].debugInfo`.

This means any service in your application can contribute to the debug output without the kernel needing to know about it:

```php
final class QueryLog implements DebuggableInterface
{
    private array $queries = [];

    public function record(string $sql, float $duration): void
    {
        $this->queries[] = ['sql' => $sql, 'duration' => $duration];
    }

    public function getDebugInfo(): array
    {
        return [
            'query_count' => count($this->queries),
            'queries'     => $this->queries,
        ];
    }
}
```

Register it as a shared service and it will appear in `$kernel->getDebugInfo()` after it has been resolved from the container at least once.
