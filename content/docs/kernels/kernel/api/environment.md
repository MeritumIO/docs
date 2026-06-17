+++
title = "Environment"
description = "Enum representing the application environment"
weight = 8
+++

# Environment

`Georgeff\Kernel\Environment`

A backed enum representing the application environment. Passed to the `Kernel` constructor and available to modules via [`ConfigurableModuleInterface::config()`](/kernel/api/configurable-module-interface/) and [`ModuleRepositoryInterface::modules()`](/kernel/api/module-repository-interface/).

## Cases

```php
enum Environment: string
{
    case Local       = 'local';       // Local development machines
    case Production  = 'production';
    case Staging     = 'staging';
    case Development = 'development'; // Remote dev/integration environments
    case Testing     = 'testing';
}
```

## Usage

```php
use Georgeff\Kernel\Environment;
use Georgeff\Kernel\Kernel;
use Georgeff\Kernel\Support\Env;

$environment = match (Env::get('APP_ENV', 'production')) {
    'local'                        => Environment::Local,
    'test', 'testing'              => Environment::Testing,
    'dev', 'develop', 'development'=> Environment::Development,
    'stage', 'staging'             => Environment::Staging,
    default                        => Environment::Production,
};

$kernel = new Kernel($environment, debug: $environment === Environment::Local);
```

`KernelInterface::getEnvironment()` returns the string value (e.g., `'local'`), not the enum case.
