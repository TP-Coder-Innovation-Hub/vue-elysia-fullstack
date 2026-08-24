# Error Handling

## What

Elysia errors follow one rule: throw or return an error value, map it to a status code in one place, and let every response — success or failure — have a predictable shape.

## Why It Matters

Ad-hoc error handling produces ad-hoc APIs: one route returns `{ error: "..." }` with 500, another returns raw text with 400, and clients write fragile per-route parsing. A typed error model keeps handlers focused on the happy path, gives the frontend one error contract, and makes logs actually useful in production.

## How It Works

### `error()` — The Standard Way to Fail

```ts
import { Elysia, error, t } from 'elysia'

new Elysia()
  .get('/tasks/:id', async ({ params }) => {
    const task = await tasks.findById(params.id)
    if (!task) return error(404, 'Task not found')
    return task
  })
```

`error()` returns a typed error value — handlers stay expression-like; no throw/catch ceremony in the happy path.

### Custom Error Codes with `onError`

```ts
// errors.ts
export class DomainError extends Error {
  constructor(
    public code: string,
    public status: number,
    message: string
  ) { super(message) }
}

export const NotFound = (what: string) =>
  new DomainError('NOT_FOUND', 404, `${what} not found`)

export const Forbidden = () =>
  new DomainError('FORBIDDEN', 403, 'You do not own this resource')
```

```ts
// index.ts
new Elysia()
  .onError(({ code, error: err, set }) => {
    if (err instanceof DomainError) {
      set.status = err.status
      return { error: { code: err.code, message: err.message } }
    }
    if (code === 'VALIDATION') {
      set.status = 422
      return { error: { code: 'VALIDATION', message: 'Invalid input' } }
    }
    console.error(err)
    set.status = 500
    return { error: { code: 'INTERNAL', message: 'Something went wrong' } }
  })
```

Every failure — thrown or returned — funnels into one response shape.

### Use It in Handlers

```ts
.delete('/tasks/:id', async ({ params, user }) => {
  const task = await tasks.findById(params.id)
  if (!task) throw NotFound('Task')
  if (task.userId !== user.id) throw Forbidden()
  await tasks.remove(task.id)
  return { deleted: true }
})
```

Business rules read like business rules; mechanics live in `onError`.

### Validation Errors for Free

Fail a `t.Schema` and Elysia emits a `VALIDATION` error with details of what failed — your Vue form maps `error.value` fields back to inputs.

```mermaid
flowchart TD
    A[handler] --> B{failure?}
    B -->|return error\\(\\)| C[onError]
    B -->|throw DomainError| C
    B -->|schema fail| C
    C --> D{code / type}
    D -->|DomainError| E[its status + shape]
    D -->|VALIDATION| F[422 + details]
    D -->|else| G[500 + generic message, logged]
```

## Common Mistakes

- **Leaking stack traces.** Return a generic message on 500; log the details server-side only.
- **200 with `{ error }` inside.** Failed requests must carry 4xx/5xx status codes — clients, proxies, and monitors depend on them.
- **String-matching error messages.** Compare `error.value.code`, not `message` text — messages change, codes should not.
- **Catching just to re-wrap.** Let unexpected errors hit `onError`; wrap only errors you add meaning to.
