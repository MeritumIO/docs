+++
title = "RepositoryInterface"
description = "Contract for model persistence: save, delete, find, findOrFail, findBy"
weight = 1
+++

# RepositoryInterface

`Meritum\Database\RepositoryInterface<T extends Model>`

The public contract for model repositories. Type-hint against this interface at injection sites rather than concrete repository classes.

## Methods

```php
public function save(Model $model): bool;
```

Inserts or updates `$model`. Returns `true` on success.

---

```php
public function delete(Model $model): bool;
```

Deletes `$model` by its primary key. Returns `true` on success.

---

```php
public function find(int|string $pk): ?T;
```

Returns the model with the given primary key, or `null` when not found.

---

```php
public function findOrFail(int|string $pk): T;
```

Returns the model with the given primary key. Throws `ModelNotFoundException` when not found.

---

```php
public function findBy(string $column, mixed $value): ?T;
```

Returns the first model where `$column` equals `$value`, or `null`.

## Usage

Define your own interface per repository that extends `RepositoryInterface`, and bind the concrete class against it:

```php
use Meritum\Database\RepositoryInterface;

interface UserRepositoryInterface extends RepositoryInterface
{
    public function findByEmail(string $email): ?User;
}
```

```php
$kernel->define(UserRepositoryInterface::class, fn($c) => new UserRepository(
    $c->get(DatabaseManagerInterface::class)
));
```

Consuming classes type-hint the specific interface:

```php
final class CreateUser
{
    public function __construct(
        private readonly UserRepositoryInterface $users
    ) {}
}
```

The concrete [`Repository`](/docs/packages/database/api/repository/) abstract class implements this interface and adds the protected query-building infrastructure.
