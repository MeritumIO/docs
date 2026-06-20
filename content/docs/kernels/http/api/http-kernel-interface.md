+++
title = "HttpKernelInterface"
description = "The primary contract for the HTTP kernel"
weight = 1
+++

# HttpKernelInterface

`Meritum\Http\HttpKernelInterface`

Extends `KernelInterface`, `RunnableKernelInterface`, and `Psr\Http\Server\RequestHandlerInterface`. `HttpKernel` is the concrete implementation.

All methods inherited from [`KernelInterface`](/docs/kernels/kernel/api/kernel-interface/) are available.

## Methods

### Routing

```php
public function addRoute(array|string $methods, string $uri, RequestHandlerInterface|string $handler): RouteInterface;
```

Registers a route. `$methods` is a single HTTP method string or an array of methods. `$handler` is either a `RequestHandlerInterface` instance or a class string to be resolved from the container at request time.

Returns a [`RouteInterface`](/docs/kernels/http/api/route-interface/) for further configuration (per-route middleware, etc.).

Throws `KernelException` if called after `boot()`.

---

### Middleware

```php
public function addMiddleware(MiddlewareInterface|string $middleware): static;
```

Adds a PSR-15 middleware to the global stack. Middleware runs in registration order. A class string is resolved from the container at request time.

Throws `KernelException` if called after `boot()`.

---

### Termination

```php
public function onTerminating(callable $callback): static;
```

Registers a callback to run after the response is emitted. Signature: `callable(ServerRequestInterface, ResponseInterface, KernelInterface): void`.

```php
public function terminate(ServerRequestInterface $request, ResponseInterface $response): void;
```

Runs all registered terminating callbacks in order. Called automatically by `run()`.

---

### Request handling

```php
public function handle(ServerRequestInterface $request): ResponseInterface;
```

Passes the request through the middleware stack and router. If an exception escapes and `ExceptionHandlerInterface` is registered in the container, it is invoked. Otherwise the exception is re-thrown.

```php
public function run(): int;
```

The main entry point. Resolves the request from globals, calls `handle()`, emits the response via SAPI, calls `terminate()`, then calls `shutdown()`. Returns `0`.
