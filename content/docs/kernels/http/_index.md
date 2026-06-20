+++
bookCollapseSection = true
weight = 2
title = "meritum/http"
description = "Module-first PSR-15 HTTP kernel for the Meritum ecosystem"
+++

# meritum/http

`meritum/http` extends `georgeff/kernel` with routing, middleware, and request handling. Everything in the [kernel guide](/docs/kernels/kernel/) applies here — modules, service definitions, tagging, decoration, and the boot lifecycle all work identically.

```
composer require meritum/http
```

## Entry Point

`HttpKernel` is the concrete implementation. Wire it up in your entry point script:

```php
use App\ModuleRepository;
use Meritum\Http\HttpKernel;
use Georgeff\Kernel\Support\Env;
use Georgeff\Kernel\Environment;

$environment = match (Env::get('APP_ENV', 'production')) {
    'local'                         => Environment::Local,
    'test', 'testing'               => Environment::Testing,
    'dev', 'develop', 'development' => Environment::Development,
    'stage', 'staging'              => Environment::Staging,
    default                         => Environment::Production,
};

$kernel = new HttpKernel($environment, debug: Env::get('APP_DEBUG', false));
$kernel->addRepository(new ModuleRepository());
$kernel->boot();
$kernel->run();
```

`run()` resolves the incoming request from globals, passes it through the middleware stack and router, emits the response, runs terminating callbacks via `terminate()`, then shuts the kernel down.

## Routing

Register routes before `boot()`, either directly on the kernel or inside a module:

```php
$kernel->addRoute('GET', '/', Handler\HomeHandler::class);
$kernel->addRoute(['GET', 'HEAD'], '/health', Handler\HealthHandler::class);
$kernel->addRoute('POST', '/users', Handler\CreateUserHandler::class);
```

The handler can be a class string (resolved from the container at request time) or a `RequestHandlerInterface` instance.

Handlers are PSR-15 `RequestHandlerInterface` implementations:

```php
use Laminas\Diactoros\Response\JsonResponse;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class HomeHandler implements RequestHandlerInterface
{
    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        return new JsonResponse(['status' => 'ok']);
    }
}
```

### Route Parameters

Capture URI segments using `{name}` placeholders:

```php
$kernel->addRoute('GET', '/users/{id}', Handler\ShowUserHandler::class);
```

Parameters are available via the route attached to the request attributes:

```php
use Meritum\Http\Routing\RouteInterface;

final class ShowUserHandler implements RequestHandlerInterface
{
    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        /** @var RouteInterface $route */
        $route = $request->getAttribute(RouteInterface::class);
        $id = $route->getArgument('id');

        return new JsonResponse(['id' => $id]);
    }
}
```

The route is also available under the `'__route__'` string key as a convenience alias for code that does not import `RouteInterface`.

### Registering Routes from a Module

Route registration inside a module requires narrowing `KernelInterface` to `HttpKernelInterface`:

```php
use Georgeff\Kernel\KernelInterface;
use Meritum\Http\HttpKernelInterface;
use Georgeff\Kernel\Module\ModuleInterface;

final class AppModule implements ModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        assert($kernel instanceof HttpKernelInterface);

        $kernel->define(Handler\HomeHandler::class, fn() => new Handler\HomeHandler());

        $kernel->addRoute('GET', '/', Handler\HomeHandler::class);
    }
}
```

## Middleware

### Global Middleware

Add PSR-15 middleware to the global stack with `addMiddleware()`. Middleware runs in registration order, wrapping every request:

```php
$kernel->addMiddleware(Middleware\CorsMiddleware::class);
$kernel->addMiddleware(Middleware\AuthMiddleware::class);
```

As with handlers, a class string is resolved from the container at request time.

### Per-Route Middleware

Attach middleware to a specific route via the `RouteInterface` returned by `addRoute()`:

```php
$kernel->addRoute('DELETE', '/users/{id}', Handler\DeleteUserHandler::class)
    ->addMiddleware(Middleware\RequireAdminMiddleware::class);
```

Per-route middleware runs after global middleware, only for matching requests.

## Exception Handling

By default, exceptions that escape the middleware stack are re-thrown. To handle them and convert them to responses, register an implementation of `ExceptionHandlerInterface` in the container:

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

Register it in a module:

```php
use Meritum\Http\Exception\ExceptionHandlerInterface;

$kernel->define(ExceptionHandlerInterface::class, fn() => new ExceptionHandler());
```

The kernel checks for `ExceptionHandlerInterface::class` in the container on every request. If present, caught exceptions are passed to it. If absent, exceptions bubble.

> `meritum/http-exception-handler` provides a production-ready implementation that integrates with the `meritum/structured-logging` pipeline. See the [http-exception-handler guide](/docs/packages/http-exception-handler/) for details.

## HTTP Exceptions

`meritum/http` ships three HTTP exception classes for common failure cases:

```php
use Meritum\Http\Exception\NotFoundHttpException;
use Meritum\Http\Exception\MethodNotAllowedHttpException;
use Meritum\Http\Exception\HttpException;

throw new NotFoundHttpException($request);
throw new MethodNotAllowedHttpException($request);
throw new HttpException($request, 'Custom message'); // 500 by default
```

All three implement `HttpExceptionInterface`, which your exception handler can check to extract the status code and title.

## Terminating Callbacks

Register callbacks to run after the response has been emitted:

```php
$kernel->onTerminating(function (ServerRequestInterface $request, ResponseInterface $response, KernelInterface $kernel): void {
    // Runs after the response is sent to the client
});
```

Multiple callbacks run in registration order. `terminate()` does not shut the kernel down — `shutdown()` is called by `run()` after all terminating callbacks have completed.

When calling `handle()` and `terminate()` directly (e.g. in tests), shutdown is your responsibility:

```php
$kernel->boot();
$response = $kernel->handle($request);
$kernel->terminate($request, $response);
$kernel->shutdown();
```

## Default Container Registrations

`HttpKernel` registers two services automatically during the `preBoot` phase:

| ID | Resolves to |
|---|---|
| `RequestHandlerInterface::class` | The router (wrapped middleware stack + route dispatch) |
| `ServerRequestInterface::class` | `ServerRequestFactory::fromGlobals()` |

Both are shared. Do not overwrite these unless you intend to replace the router or request resolution entirely.
