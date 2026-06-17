+++
title = "ExitCode"
description = "Enum representing command exit codes"
weight = 8
+++

# ExitCode

`Meritum\Cli\ExitCode`

Backed enum returned by `Command::__invoke()`. The integer value is passed back to the shell as the process exit code.

| Case | Value | Meaning |
|---|---|---|
| `ExitCode::Success` | `0` | Command completed successfully |
| `ExitCode::Error` | `1` | Command failed |

```php
public function __invoke(SageStyleInterface $io): ExitCode
{
    if (!$this->canProceed()) {
        $io->error('Cannot proceed.');
        return ExitCode::Error;
    }

    return ExitCode::Success;
}
```

The CLI kernel always returns `ExitCode::Error` (1) when an exception is caught and handled by the `ExceptionHandlerInterface`.
