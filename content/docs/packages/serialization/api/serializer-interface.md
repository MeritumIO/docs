+++
title = "SerializerInterface"
description = "Transforms a domain object into a serialized array"
weight = 1
+++

# SerializerInterface

`Meritum\Serialization\SerializerInterface`

Transforms a single domain object into an `array<string, mixed>`. Passed to `Item` or `Collection` at construction time to define how each record in that resource is serialized.

## Methods

```php
/** @return array<string, mixed> */
public function serialize(mixed $data): array;
```

Receives the raw data object and returns its array representation. For collections, this is called once per element.

## Example

```php
use Meritum\Serialization\SerializerInterface;

final class UserSerializer implements SerializerInterface
{
    public function serialize(mixed $data): array
    {
        return [
            'id'         => $data->id,
            'name'       => $data->name,
            'email'      => $data->email,
            'created_at' => $data->createdAt->format('c'),
        ];
    }
}
```
