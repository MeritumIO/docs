+++
title = "CursorInterface"
description = "Cursor-based pagination contract for collections"
weight = 6
+++

# CursorInterface

`Meritum\Serialization\Pagination\CursorInterface`

Provides cursor-based pagination metadata to a [`Collection`](/docs/packages/serialization/api/collection/). Implement this interface and call `Collection::setCursor()`.

## Methods

```php
public function getPrevious(): ?string;
```

The cursor token for the previous page, or `null` if this is the first page.

```php
public function getNext(): ?string;
```

The cursor token for the next page, or `null` if this is the last page.

```php
public function getPerPage(): int;
```

The number of items per page.

## Output keys

When a cursor is attached to a collection, these keys appear in the envelope alongside `data`:

| Key | Source |
|---|---|
| `limit` | `getPerPage()` |
| `previous` | `getPrevious()` |
| `next` | `getNext()` |

## Example

```php
use Meritum\Serialization\Pagination\CursorInterface;

final class UserCursor implements CursorInterface
{
    public function __construct(
        private readonly ?string $previous,
        private readonly ?string $next,
        private readonly int $perPage
    ) {}

    public function getPrevious(): ?string { return $this->previous; }
    public function getNext(): ?string     { return $this->next; }
    public function getPerPage(): int      { return $this->perPage; }
}

$resource = new Collection($users, $serializer);
$resource->setCursor(new UserCursor(previous: null, next: 'eyJpZCI6MjV9', perPage: 25));
```
