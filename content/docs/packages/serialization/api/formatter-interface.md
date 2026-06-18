+++
title = "FormatterInterface"
description = "Formats a resource into an envelope using the configured strategy"
weight = 2
+++

# FormatterInterface

`Meritum\Serialization\FormatterInterface`

Converts an `Item` or `Collection` resource into an [`EnvelopeInterface`](/docs/packages/serialization/api/envelope-interface/) using the configured [`StrategyInterface`](/docs/packages/serialization/api/strategy-interface/). Registered by `SerializationModule` as `FormatterInterface::class`.

## Methods

```php
public function format(ResourceInterface $resource): EnvelopeInterface;
```

Dispatches to the strategy's `item()` or `collection()` method based on the resource type. Throws `InvalidArgumentException` if the resource is neither an `ItemInterface` nor a `CollectionInterface`.

## Example

```php
use Meritum\Serialization\Resource\Item;
use Meritum\Serialization\FormatterInterface;

final class ShowUserHandler implements RequestHandlerInterface
{
    public function __construct(
        private readonly FormatterInterface $formatter,
        private readonly UserSerializer $serializer,
        private readonly UserRepository $users
    ) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        $user = $this->users->find($request->getAttribute('id'));

        $envelope = $this->formatter->format(new Item($user, $this->serializer));

        return new JsonResponse($envelope);
    }
}
```
