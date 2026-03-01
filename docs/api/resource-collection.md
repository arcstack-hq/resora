# ResourceCollection

Handles transformation of resource collections.

Supports:

- Pagination metadata
- Cursor metadata
- Resource mapping via `collects`

---

## Constructor

```ts
new ResourceCollection(resource, response?)
```

## Mapping Items

You can define a resource transformer:

```ts
class UserCollection extends ResourceCollection<User[]> {
  collects = UserResource;
}
```

Each item will be transformed using `UserResource`.

## Metadata Handling

If the resource contains:

```ts
{
  data: [],
  pagination: {},
  cursor: {}
}
```

Resora automatically adds:

```json
{
  "data": [],
  "meta": {},
  "cursor": {}
}
```
