+++
title = "CursorPaginator"
description = "Keyset-based pagination result with opaque cursor tokens"
weight = 5
+++

# CursorPaginator

`Meritum\Database\Support\CursorPaginator<T extends Model>`

Returned by `Repository::cursor()`. Uses keyset pagination on the primary key rather than offset-based counting, which keeps query cost constant regardless of how deep into the result set the cursor is.

## Properties

```php
public readonly int     $perPage;
public readonly ?string $nextCursor;
public readonly ?string $previousCursor;
```

| Property | Description |
|---|---|
| `$perPage` | Page size as requested |
| `$nextCursor` | Opaque token for the next page, or `null` on the last page |
| `$previousCursor` | Opaque token for the previous page, or `null` on the first page |

Cursor tokens are opaque base64url-encoded strings. Do not parse them — pass them back to `cursor()` as-is.

## Methods

```php
public function hasMorePages(): bool;
```

Returns `true` when `$nextCursor` is not `null`.

---

```php
public function hasPreviousPages(): bool;
```

Returns `true` when `$previousCursor` is not `null`.

---

```php
public function collection(): Collection<T>;
```

Returns the hydrated model collection for the current page.

## Usage

Pass `null` on the first request. Pass the `nextCursor` or `previousCursor` from the previous result to navigate:

```php
// First page
$page = $userRepository->listCursor(perPage: 20, cursor: null);

// Next page
$page = $userRepository->listCursor(perPage: 20, cursor: $page->nextCursor);

// Previous page
$page = $userRepository->listCursor(perPage: 20, cursor: $page->previousCursor);
```

Expose `nextCursor` and `previousCursor` in your API response so clients can navigate without building their own tokens.
