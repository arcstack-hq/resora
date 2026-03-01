# Using Resora Outside H3/Express (Non-Connect Frameworks)

Resora works as a pure transformation layer, even when your framework does **not** expose a Connect-style response object.

In these frameworks, use Resora to build a JSON-safe payload, then return or send it with your framework’s native response API.

## Core Pattern

```ts
import { Resource } from 'resora';

const body = new Resource(user).getBody();

// send `body` using your framework's response mechanism
```

This approach works everywhere because it does not rely on `.response()` transport binding.

## Why this works

Resora separates:

- **Transformation**: resource shape, metadata, conditional attributes
- **Transport**: headers/status/cookies at framework response layer

When Connect-style response objects are unavailable, keep transport in your framework and use Resora for transformation only.

## Fastify Example (Current Recommended Pattern)

```ts
import { Resource } from 'resora';

fastify.get('/users/:id', async (request, reply) => {
  const user = await findUser(request.params.id);

  const body = new Resource(user).withMeta({ traceId: request.id }).getBody();

  return reply.code(200).send(body);
});
```

## NestJS Example (Current Recommended Pattern)

### Return value style

```ts
import { Resource } from 'resora'

@Get(':id')
async findOne(@Param('id') id: string) {
  const user = await this.usersService.findOne(id)

  return new Resource(user)
    .withMeta({ source: 'users-controller' })
    .getBody()
}
```

### `@Res()` style

```ts
import { Resource } from 'resora'

@Get(':id')
async findOne(@Param('id') id: string, @Res() res: any) {
  const user = await this.usersService.findOne(id)

  const body = new Resource(user).getBody()

  return res.status(200).json(body)
}
```

## Collections and Pagination in Non-Connect Frameworks

```ts
import { ResourceCollection } from 'resora';

const body = new ResourceCollection({
  data: users,
  pagination: {
    currentPage: 1,
    lastPage: 10,
    total: 100,
    perPage: 10,
    path: '/users',
  },
}).getBody();
```

Then send `body` with your framework's normal response API.

## Conditional Attributes Still Work

```ts
class UserResource extends Resource {
  data() {
    return {
      id: this.id,
      email: this.whenNotNull(this.email),
      role: this.when(this.isAdmin, 'admin'),
    };
  }
}
```

All conditional helpers work the same because they are transformation features, not transport features.

## Important Note About `.response()`

Today, `.response()` and transport methods are designed for H3/Express/Connect-style response objects.

For non-Connect frameworks, prefer `.getBody()` and framework-native sending, as shown above.
