+++
title = "ConfigurableModuleInterface"
description = "Module contract for contributing to the shared kernel config"
weight = 4
+++

# ConfigurableModuleInterface

`Georgeff\Kernel\Module\ConfigurableModuleInterface`

Extends [`ModuleInterface`](/kernel/api/module-interface/). Adds a `config()` method for contributing to the kernel's shared configuration array.

## Methods

```php
public function config(Environment $env): array;
```

Returns an `array<string, mixed>` of configuration values. Called during the `moduleLoad` boot phase, before `register()`. Results from all configurable modules are merged together and registered in the container as `'kernel.config'`.

The `Environment` parameter lets you return environment-specific values.

## Example

```php
use Georgeff\Kernel\Environment;
use Georgeff\Kernel\Support\Env;
use Georgeff\Kernel\KernelInterface;
use Georgeff\Kernel\Module\ConfigurableModuleInterface;

final class DatabaseModule implements ConfigurableModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(Connection::class, fn($c) => new Connection(
            $c->get('kernel.config')['database']
        ))->share();
    }

    public function config(Environment $env): array
    {
        return [
            'database' => [
                'host'     => Env::get('DB_HOST', '127.0.0.1'),
                'port'     => (int) Env::get('DB_PORT', 5432),
                'name'     => Env::get('DB_NAME', 'app'),
                'user'     => Env::get('DB_USER', 'app'),
                'password' => Env::get('DB_PASSWORD', ''),
            ],
        ];
    }
}
```

> `kernel.config` is only available in the container after `boot()`. Accessing it before boot throws.
