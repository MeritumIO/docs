+++
title = "ExceptionHandlerInterface"
description = "Converts unhandled exceptions into HTTP responses"
weight = 2
+++

# ExceptionHandlerInterface

`Meritum\Http\Exception\ExceptionHandlerInterface`

Converts exceptions that escape the middleware stack into HTTP responses. Register an implementation as `ExceptionHandlerInterface::class` in the container to activate it. If no implementation is registered, exceptions are re-thrown.

## Methods

```php
public function handle(Throwable $e, ServerRequestInterface $request): ResponseInterface;
```

Called by the kernel when an exception escapes the middleware stack. Should return a response for any throwable — never throw from this method.

## Example

```php
use Throwable;
use Laminas\Diactoros\Response\JsonResponse;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Meritum\Http\Exception\ExceptionHandlerInterface;
use Meritum\Http\Exception\HttpExceptionInterface;

final class ExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $e, ServerRequestInterface $request): ResponseInterface
    {
        if ($e instanceof HttpExceptionInterface) {
            return new JsonResponse(
                ['error' => $e->getTitle()],
                $e->getStatusCode()
            );
        }

        return new JsonResponse(['error' => 'Internal Server Error'], 500);
    }
}
```

Register in a module:

```php
use Meritum\Http\Exception\ExceptionHandlerInterface;

$kernel->define(ExceptionHandlerInterface::class, fn() => new ExceptionHandler());
```
