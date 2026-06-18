+++
title = "Severity"
description = "Enum representing the five severity levels used by DomainException"
weight = 7
+++

# Severity

`Meritum\StructuredLogging\Severity`

Backed enum used by [`DomainException`](/docs/packages/structured-logging/api/domain-exception/) to declare the exception's severity. Maps to the five-bucket severity scheme used by `meritum/logger`.

| Case | Value | When to use |
|---|---|---|
| `Severity::Critical` | `'critical'` | System is in a critical state; immediate attention required |
| `Severity::Error` | `'error'` | An operation failed; normal flow is disrupted |
| `Severity::Warning` | `'warning'` | Something unexpected occurred but the operation continued |
| `Severity::Info` | `'info'` | Informational domain event worth recording |
| `Severity::Debug` | `'debug'` | Low-level diagnostic detail |

`ExceptionReporter` passes `$severity->value` directly to `LoggerInterface::log()` as the log level.
