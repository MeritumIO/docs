+++
title = "RouteInterface"
description = "Represents a registered route and its configuration"
weight = 3
+++

# RouteInterface

`Meritum\Http\Routing\RouteInterface`

Returned by `HttpKernelInterface::addRoute()`. Used to attach per-route middleware and to read route parameters inside a handler.

## Configuration methods

```php
public function addMiddleware(MiddlewareInterface|string $middleware): static;
```

Attaches middleware to this route only. Per-route middleware runs after global middleware, for matching requests only. Can be chained.

```php
public function hasMiddleware(): bool;
public function getMiddleware(): iterable;
```

---

## Route parameter methods

```php
public function getArguments(): array;
```

Returns all captured route parameters as `array<string, string>`.

```php
public function getArgument(string $name, ?string $default = null): ?string;
```

Returns a single captured parameter by name, or `$default` if not present.

---

## Other accessors

```php
public function getMethods(): array;   // non-empty-list<string>
public function getPath(): string;
public function getHandler(): RequestHandlerInterface|string;
```

---

## Accessing the route in a handler

The matched route is attached to the request as an attribute:

```php
use Meritum\Http\Routing\RouteInterface;
use Psr\Http\Message\ServerRequestInterface;

/** @var RouteInterface $route */
$route = $request->getAttribute('__route__');

$id   = $route->getArgument('id');
$page = $route->getArgument('page', '1');
```
