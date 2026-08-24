# Testing with Bun and Eden

## What

Bun ships a test runner (`bun test`) with a Jest-like API. Use it for three layers: unit tests for logic, API tests that call Elysia handlers via Eden Treaty, and component tests for Vue.

## Why It Matters

A fullstack app without tests breaks silently — a schema change on one side surfaces as a broken screen on the other. The highest-value tests for this stack are the API-level ones: request in, response out, database state asserted. They catch validation bugs, scoping mistakes (user A reading user B's data), and regressions in one place, without mocking every internal.

## How It Works

### Test the API Through Eden

```ts
// tests/tasks.test.ts
import { beforeAll, afterAll, describe, expect, it } from 'bun:test'
import { treaty } from '@elysiajs/eden'
import mongoose from 'mongoose'
import { app } from '../src/index'
import type { App } from '../src/index'

const api = treaty<App>(app)     // in-process — no running server needed

let token = ''

beforeAll(async () => {
  await mongoose.connect(process.env.DATABASE_URL!)
  const { data } = await api.auth.login.post({
    email: 'test@example.com',
    password: 'password123'
  })
  token = data!.token
})

afterAll(async () => {
  await mongoose.connection.dropDatabase()
  await mongoose.disconnect()
})

describe('POST /tasks', () => {
  it('creates a task for the current user', async () => {
    const { data, status } = await api.tasks.post(
      { title: 'Write tests', priority: 'high' },
      { headers: { authorization: `Bearer ${token}` } }
    )

    expect(status).toBe(201)
    expect(data?.title).toBe('Write tests')
  })

  it('rejects an invalid body with 422', async () => {
    const { status } = await api.tasks.post(
      { title: '' },
      { headers: { authorization: `Bearer ${token}` } }
    )
    expect(status).toBe(422)
  })

  it('does not let another user read my task', async () => {
    const { data } = await api.tasks.post(
      { title: 'Private task' },
      { headers: { authorization: `Bearer ${token}` } }
    )
    const other = await api.auth.login.post({
      email: 'other@example.com', password: 'password123'
    })

    const res = await api.task({ id: data!.id }).get(undefined, {
      headers: { authorization: `Bearer ${other.data!.token}` }
    })
    expect(res.status).toBe(404)
  })
})
```

Type safety applies in tests too — bad request shapes fail to compile.

### Unit-Test Pure Logic

```ts
// tests/ownership.test.ts
import { describe, expect, it } from 'bun:test'
import { canModify } from '../src/domain/ownership'

describe('canModify', () => {
  it('allows the owner', () => {
    expect(canModify({ id: 'u1' }, { userId: 'u1' })).toBe(true)
  })
})
```

### Vue Component Tests

For components, use Vitest (shares Vite config) with Vue Test Utils:

```ts
// frontend/src/components/__tests__/TaskCard.spec.ts
import { mount } from '@vue/test-utils'
import TaskCard from '../TaskCard.vue'

it('renders the task title', () => {
  const wrapper = mount(TaskCard, {
    props: { task: { id: '1', title: 'Write tests', completed: false } }
  })
  expect(wrapper.text()).toContain('Write tests')
})
```

| Layer | Runner | What it proves |
|-------|--------|----------------|
| Logic | `bun test` | Rules and helpers |
| API | `bun test` + Eden | Routes, validation, scoping |
| Components | Vitest + Test Utils | Rendering and events |

Run everything: `bun test` in the server, `bunx vitest run` in the frontend.

## Common Mistakes

- **Testing a running server via URL.** Import the `app` instance and Treaty it in-process — no port juggling, faster, and `.listen()` stays out of test runs.
- **Sharing a database across suites.** Each test run should get a fresh (or dropped) database —CI and local results must match.
- **Mocking Mongoose everywhere.** Assert real database state for scoping and persistence bugs; mock only third-party APIs (email, payments).
- **Chasing 100% coverage.** Cover the API contract and security rules first — those failures are the expensive ones.
