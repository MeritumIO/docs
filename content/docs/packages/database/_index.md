+++
bookCollapseSection = true
weight = 2
title = "meritum/database"
description = "Model and repository layer for the Meritum ecosystem"
+++

# meritum/database

`meritum/database` provides a model and repository layer built on `georgeff/database`. Models hold typed, cast-aware attributes; repositories handle persistence and querying against a `DatabaseManagerInterface`.

```
composer require meritum/database
```

## Module Registration

Add `DatabaseModule` to your module repository:

```php
use Meritum\Database\DatabaseModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new DatabaseModule(),
            // ...
        ];
    }
}
```

The module reads connection configuration from environment variables and registers `DriverInterface`, `ConnectionManagerInterface`, and `DatabaseManagerInterface` in the container. Inject `DatabaseManagerInterface` into your repositories.

## Configuration

Set these environment variables before boot:

### PostgreSQL

```
DB_DRIVER=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=myapp
DB_USERNAME=app
DB_PASSWORD=secret
DB_PGSQL_SCHEMA=public
DB_PGSQL_SSL_MODE=prefer
```

### MySQL

```
DB_DRIVER=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=app
DB_PASSWORD=secret
DB_MYSQL_CHARSET=utf8mb4
```

### SQLite

```
DB_DRIVER=sqlite
DB_DATABASE=/path/to/database.sqlite
```

### Read replicas

`DB_READ_HOSTS` accepts a JSON array of read replica hostnames (e.g. `["replica1.db", "replica2.db"]`). When set, select queries are routed to a read replica. `DB_STICKY_WRITE=true` (default) routes reads to the write connection for the remainder of the request after any write, avoiding read-after-write lag from replication delay.

## Defining a Model

Extend `Model` and define your attributes as PHP 8.4 property hooks backed by `getAttribute()` and `setAttribute()`:

```php
use Meritum\Database\Model;

final class User extends Model
{
    protected string $table = 'users';

    protected array $casts = [
        'email_verified_at' => 'datetime',
        'is_active'         => 'bool',
    ];

    public string $id {
        get => $this->getAttribute('id');
    }

    public string $name {
        get => $this->getAttribute('name');
        set => $this->setAttribute('name', $value);
    }

    public string $email {
        get => $this->getAttribute('email');
        set => $this->setAttribute('email', $value);
    }

    public ?\DateTimeImmutable $emailVerifiedAt {
        get => $this->getAttribute('email_verified_at');
    }

    public bool $isActive {
        get => $this->getAttribute('is_active');
        set => $this->setAttribute('is_active', $value);
    }
}
```

`$table` must be set. `$primaryKey` defaults to `'id'`. Timestamps (`created_at` / `updated_at`) are managed automatically unless you set `$timestamps = false`.

## Primary Keys

By default models use string (UUID v4) primary keys. The repository auto-generates a UUID when saving a new model with no primary key value set.

```php
// String UUID (default) — $primaryKeyType = 'string', $incrementing = false
protected string $primaryKey     = 'id';
protected string $primaryKeyType = 'string';
protected bool   $incrementing   = false;
```

For auto-increment integer primary keys:

```php
protected string $primaryKey     = 'id';
protected string $primaryKeyType = 'int';
protected bool   $incrementing   = true;
```

To use a custom UUID version, override `generateUuid()` in your repository.

## Attribute Casts

Declare casts in `$casts` to convert between PHP types and database storage:

```php
protected array $casts = [
    'age'        => 'int',
    'score'      => 'float',
    'is_active'  => 'bool',
    'metadata'   => 'json',
    'created_at' => 'datetime',
    'birthday'   => 'date',
    'expires_at' => 'timestamp',
];
```

| Cast | PHP type on read | Stored as |
|---|---|---|
| `int` | `int` | int |
| `float` | `float` | float |
| `string` | `string` | string |
| `bool` | `bool` | bool |
| `json` | `array` (decoded) | JSON string |
| `datetime` | `DateTimeImmutable` (with time) | `Y-m-d H:i:s` string |
| `date` | `DateTimeImmutable` (midnight) | `Y-m-d` string |
| `timestamp` | `DateTimeImmutable` | Unix timestamp integer |

`toArray()` formats `DateTimeImmutable` values as ISO 8601 strings.

## Accessors and Mutators

Override `accessors()` and `mutators()` to apply transformations on read and write:

```php
protected function accessors(): array
{
    return [
        'name' => fn(string $value) => ucwords($value),
    ];
}

protected function mutators(): array
{
    return [
        'email' => fn(string $value) => strtolower($value),
    ];
}
```

Accessors run after casting on read. Mutators run before casting on write.

## Relations

Models carry a protected relation bag for attaching related data that has been loaded separately. There is no lazy loading — the caller fetches related records independently and attaches them to the model.

Expose relations through typed public wrapper methods on the concrete model:

```php
final class EventLog extends Model
{
    protected string $table = 'event_logs';

    public function setEvent(Event $event): void
    {
        $this->setRelation('event', $event);
    }

    public function getEvent(): Event
    {
        /** @var Event */
        return $this->getRelation('event');
    }

    public function hasEvent(): bool
    {
        return $this->hasRelation('event');
    }
}
```

Attach the relation in a handler after running both queries independently:

```php
$log   = $logRepository->findOrFail($id);
$event = $eventRepository->findOrFail($log->eventId);

$log->setEvent($event);
```

Relations are merged into `toArray()` automatically. The key passed to `setRelation()` becomes the key in the serialized output:

```json
{
    "id": "...",
    "event_id": "...",
    "event": { "id": "...", "name": "PHP Conference" }
}
```

The relation bag accepts any JSON-safe value — a model, a collection, a paginator, a scalar, or an array. Object values must implement `JsonSerializable`; passing anything else throws a `LogicException`. The typed wrapper method on the concrete model is the actual type contract — the base bag is intentionally untyped so it does not restrict what sources or shapes of data can be attached.

## Defining a Repository

Define an interface for your repository that extends `RepositoryInterface`, then implement it by extending `Repository`:

```php
use Meritum\Database\RepositoryInterface;

interface UserRepositoryInterface extends RepositoryInterface
{
    public function findByEmail(string $email): ?User;
}
```

```php
use Meritum\Database\Repository;
use Georgeff\Database\Contract\DatabaseManagerInterface;

final class UserRepository extends Repository implements UserRepositoryInterface
{
    protected function getModelClass(): string
    {
        return User::class;
    }

    public function findByEmail(string $email): ?User
    {
        $this->query()->where('email', $email);

        return $this->first();
    }
}
```

Register it in a module bound to your own interface:

```php
$kernel->define(UserRepositoryInterface::class, fn($c) => new UserRepository(
    $c->get(DatabaseManagerInterface::class)
));
```

Consuming classes type-hint `UserRepositoryInterface`, not the concrete class.

## Built-in Finders

These methods are available on every repository:

```php
// Save (insert or update)
$user = new User();
$user->name  = 'Jane Smith';
$user->email = 'jane@example.com';
$repository->save($user); // returns bool

// Delete
$repository->delete($user); // returns bool

// Find by primary key
$user = $repository->find('uuid-here');      // returns ?User
$user = $repository->findOrFail('uuid-here'); // returns User or throws ModelNotFoundException

// Find by any column
$user = $repository->findBy('email', 'jane@example.com'); // returns ?User
```

`save()` inserts when `$model->isNew()` and updates otherwise. Timestamps and UUID generation are automatic.

## Custom Queries

Use the protected `query()` method to build a query, then execute it with `first()`, `get()`, or `count()`. `query()` returns a `SelectInterface` from `georgeff/database`; the execute methods are on the repository itself:

```php
public function findByEmail(string $email): ?User
{
    $this->query()->where('email', $email);

    return $this->first();
}

public function findActive(): Collection
{
    $this->query()->where('is_active', true)->orderBy('name', 'ASC');

    return $this->get();
}

public function countActive(): int
{
    $this->query()->where('is_active', true);

    return $this->count();
}
```

`query()` resets the internal query state each time it is called — always call it at the start of a new query chain.

## Scopes

Register reusable query constraints with `addScope()` to apply them automatically on every query:

```php
public function __construct(DatabaseManagerInterface $db)
{
    parent::__construct($db);

    $this->addScope('active', function (SelectInterface $query): void {
        $query->where('is_active', true);
    });
}
```

Bypass scopes when needed:

```php
// Bypass a specific scope
$this->withoutScope('active')->query()-> ...

// Bypass all scopes
$this->withoutScopes()->query()-> ...
```

Scope bypasses are reset after each query execution.

## Pagination

### Offset pagination

```php
public function listByDate(int $page, int $perPage): Paginator
{
    $this->query()->orderBy('created_at', 'DESC');

    return $this->paginate($perPage, $page);
}
```

`Paginator` carries the collection plus `total`, `perPage`, `currentPage`, `lastPage`, `from`, and `to`.

### Cursor pagination

```php
public function cursorPaginate(int $perPage, ?string $cursor = null): CursorPaginator
{
    $this->query()->where('is_active', true);

    return $this->cursor($perPage, $cursor);
}
```

Cursor pagination uses the primary key for ordering and keyset-based pagination. `CursorPaginator` carries the collection plus opaque `nextCursor` and `previousCursor` tokens. Pass the next/previous token back in the following request.

## Collection

`get()` and the paginator's `collection()` method return a typed `Collection<T>`. See the [Collection API page](/docs/packages/database/api/collection/) for the full method list.

```php
$users = $repository->findActive();

foreach ($users as $user) {
    echo $user->name;
}

$admins = $users->filter(fn(User $u) => $u->role === 'admin');
```
