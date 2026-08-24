# Why Vue

## What

Vue is a progressive JavaScript framework for building user interfaces. Progressive means you can adopt it incrementally — enhance a page with one component, or build a full single-page application with routing, state management, and a build toolchain.

## Why It Matters

Fullstack development is context-switching between frontend and backend. Vue reduces that cost: the Composition API with `<script setup>` is plain TypeScript, the templates stay close to HTML, and the ecosystem (Vite, Vue Router, Pinia) is first-party and coherent — one team designs the whole stack. Paired with Elysia, both sides of your app share one language and one type system, connected end-to-end by Eden Treaty.

## How It Works

### The Component Model

A Vue component is a `.vue` file with three blocks:

```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">Clicked {{ count }} times</button>
</template>

<style scoped>
button { font-weight: 600 }
</style>
```

`<script setup>` runs once when the component initializes; everything it declares is usable in the template. `scoped` styles apply only to this component.

### Progressive by Design

| Adoption level | What you write |
|---------------|----------------|
| Enhance a page | One component, no build step |
| Widget in an app | Several components via CDN |
| SPA | Vite project + Router + Pinia |

The same component syntax works at every level — no rewrite as you scale.

### The Ecosystem Is First-Party

| Tool | Role |
|------|------|
| Vite | Dev server and bundler (also used by Elysia projects) |
| Vue Router | Client-side routing |
| Pinia | State management |
| VueUse | Composable utility library |

Because all are maintained by the Vue team, they share conventions and upgrade together.

### Who Uses Vue

Alibaba, GitLab, Nintendo, Adobe, and Stack Overflow (design system). In the TypeScript-frontend market, Vue and React dominate job postings — the concepts transfer between them.

## Common Mistakes

- **Starting with Vue 2 tutorials.** Vue 3 + Composition API + `<script setup>` is the current standard; Options API knowledge is a bonus, not the goal.
- **Skipping Vite.** The `vue` global CDN build has no SFC compilation. Scaffold with `bun create vue` from day one.
- **Comparing frameworks instead of shipping.** Vue vs React matters far less than TypeScript fundamentals, HTTP, and data modeling — skills this path teaches through Vue.
