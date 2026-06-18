+++
title = "EnvelopeInterface"
description = "Final output produced by the formatter"
weight = 5
+++

# EnvelopeInterface

`Meritum\Serialization\EnvelopeInterface`

The formatted output returned by `FormatterInterface::format()`. Implements `\JsonSerializable`, so it can be passed directly to a JSON response.

## Methods

```php
/** @return array<mixed> */
public function toArray(): array;
```

Returns the formatted data as an array. Pass this to your response object.

## Example

```php
$envelope = $this->formatter->format($resource);

// Pass directly — EnvelopeInterface implements JsonSerializable
return new JsonResponse($envelope);

// toArray() is available if you need the raw array
$data = $envelope->toArray();
```
