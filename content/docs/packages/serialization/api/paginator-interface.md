+++
title = "PaginatorInterface"
description = "Offset-based pagination contract for collections"
weight = 7
+++

# PaginatorInterface

`Meritum\Serialization\Pagination\PaginatorInterface`

Provides offset-based pagination metadata to a [`Collection`](/docs/packages/serialization/api/collection/). Implement this interface and call `Collection::setPaginator()`.

## Methods

```php
public function getCurrentPage(): int;
public function getLastPage(): int;
public function getTotal(): int;
public function getCount(): int;
public function getPerPage(): int;
```

## Output keys

When a paginator is attached to a collection, these keys appear in the envelope alongside `data`:

| Key | Source |
|---|---|
| `total` | `getTotal()` |
| `count` | `getCount()` |
| `limit` | `getPerPage()` |
| `current` | `getCurrentPage()` |
| `last` | `getLastPage()` |

## Example

```php
use Meritum\Serialization\Pagination\PaginatorInterface;

final class UserPaginator implements PaginatorInterface
{
    public function __construct(
        private readonly int $total,
        private readonly int $count,
        private readonly int $perPage,
        private readonly int $currentPage,
        private readonly int $lastPage
    ) {}

    public function getTotal(): int       { return $this->total; }
    public function getCount(): int       { return $this->count; }
    public function getPerPage(): int     { return $this->perPage; }
    public function getCurrentPage(): int { return $this->currentPage; }
    public function getLastPage(): int    { return $this->lastPage; }
}

$resource = new Collection($users, $serializer);
$resource->setPaginator(new UserPaginator(
    total: 143,
    count: 25,
    perPage: 25,
    currentPage: 2,
    lastPage: 6
));
```
