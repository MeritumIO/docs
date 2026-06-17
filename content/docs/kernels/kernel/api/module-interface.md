+++
title = "ModuleInterface"
description = "Base contract for all kernel modules"
weight = 3
+++

# ModuleInterface

`Georgeff\Kernel\Module\ModuleInterface`

The base contract for all modules. Every module must implement `register()`. See also [`ConfigurableModuleInterface`](/kernel/api/configurable-module-interface/) and [`BootableModuleInterface`](/kernel/api/bootable-module-interface/) for extended contracts.

## Methods

```php
public function register(KernelInterface $kernel): void;
```

Called during the `moduleRegistration` boot phase. Use this method to register services, tags, decorators, and any other kernel configuration.

## Example

```php
use Georgeff\Kernel\KernelInterface;
use Georgeff\Kernel\Module\ModuleInterface;

final class CacheModule implements ModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(CacheInterface::class, fn() => new RedisCache())
            ->share();

        $kernel->define(CacheWarmer::class, fn($c) => new CacheWarmer(
            $c->get(CacheInterface::class)
        ));
    }
}
```
