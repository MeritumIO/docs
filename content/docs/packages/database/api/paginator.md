+++
title = "Paginator"
description = "Offset-based pagination result carrying a collection and page metadata"
weight = 4
+++

# Paginator

`Meritum\Database\Support\Paginator<T extends Model>`

Returned by `Repository::paginate()`. Carries the page's model collection alongside metadata for building page navigation.

## Properties

```php
public readonly int $total;
public readonly int $perPage;
public readonly int $currentPage;
public readonly int $lastPage;
public readonly int $from;
public readonly int $to;
```

| Property | Description |
|---|---|
| `$total` | Total number of matching rows across all pages |
| `$perPage` | Rows per page as requested |
| `$currentPage` | The page number that was fetched (1-based) |
| `$lastPage` | The highest valid page number (`ceil($total / $perPage)`) |
| `$from` | Row offset of the first item on this page (1-based) |
| `$to` | Row offset of the last item on this page (1-based) |

## Methods

```php
public function hasMorePages(): bool;
```

Returns `true` when `$currentPage < $lastPage`.

---

```php
public function collection(): Collection<T>;
```

Returns the hydrated model collection for the current page.

## Example

```php
$paginator = $userRepository->listByStatus('active', page: 2, perPage: 20);

foreach ($paginator->collection() as $user) {
    // ...
}

echo "Page {$paginator->currentPage} of {$paginator->lastPage}";
echo "Showing {$paginator->from}–{$paginator->to} of {$paginator->total}";
```
