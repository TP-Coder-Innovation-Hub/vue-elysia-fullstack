# State Management with Pinia

## What

Pinia is Vue's official state management library: typed stores that hold shared domain state — user session, server data, UI-wide flags — outside the component tree.

## Why It Matters

Props and events work for parent-child flows, but session state used by the navbar, the router guard, and five pages is not "owned" by any of them. A Pinia store gives that state one addressable home with devtools tracking, typed access, and testability. It also fits an API-backed mental model: stores are where server state lands after Eden Treaty calls.

## How It Works

### Define a Store

```ts
// stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useAuthStore = defineStore('auth', () => {
  // state
  const token = ref<string | null>(localStorage.getItem('token'))
  const user = ref<User | null>(null)

  // getters
  const isAuthenticated = computed(() => token.value !== null)

  // actions
  async function login(email: string, password: string) {
    const { token: t, user: u } = await api.auth.login({ email, password })
    token.value = t
    user.value = u
    localStorage.setItem('token', t)
  }

  function logout() {
    token.value = null
    user.value = null
    localStorage.removeItem('token')
  }

  return { token, user, isAuthenticated, login, logout }
})
```

This setup-store style is plain Composition API — `ref` is state, `computed` is getters, functions are actions.

### Consume Anywhere

```vue
<script setup lang="ts">
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()   // call inside setup
</script>

<template>
  <nav v-if="auth.isAuthenticated">{{ auth.user?.email }}</nav>
</template>
```

In the router guard:

```ts
const auth = useAuthStore()   // after pinia is installed
if (to.meta.requiresAuth && !auth.isAuthenticated) { ... }
```

### What Belongs Where

| State | Where |
|-------|-------|
| Input value of one form | Component `ref` |
| Modal open/closed for one page | Component `ref` |
| Session, current user | Pinia store |
| Tasks fetched from the API | Pinia store (or composable + store) |
| Server data with caching/invalidation | Composable over a store, or a query library |

Rule: component state is private; store state is shared by name.

### Multiple Stores Over One God Store

Split by domain — `auth`, `tasks`, `notifications`. Stores can import and call each other (`useTasksStore()` inside an action), so cross-domain workflows stay explicit.

```mermaid
flowchart TD
    A[Component] -->|login\(\)| B[auth store]
    B -->|Eden Treaty| C[Elysia API]
    C -->|user| B
    A -->|read user| B
    D[router guard] -->|isAuthenticated| B
```

## Common Mistakes

- **Replacing server state with local copies.** One canonical shape per entity in the store; components derive views from it.
- **Storing derived values.** `fullName` should be a `computed` getter, not a second `ref` kept in sync by hand.
- **Calling `useAuthStore()` at module top-level.** Pinia is not installed yet. Call it inside setup, actions, or after app install.
- **No persistence strategy.** A page refresh resets stores — persist what matters (token) to `localStorage` explicitly, and rehydrate on startup.
