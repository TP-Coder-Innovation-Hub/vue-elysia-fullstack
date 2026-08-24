# Template Syntax

## What

Vue templates are HTML enhanced with declarative bindings: interpolations render values, and directives — special attributes starting with `v-` — reactively bind the DOM to your component state.

## Why It Matters

The template is where your data becomes the interface users see. Vue's template syntax keeps rendering declarative: you describe *what* the UI should look like for a given state, and Vue updates the DOM when state changes. Master the handful of core directives and 90% of day-to-day UI work is covered.

## How It Works

### Interpolation

Double braces render text:

```vue
<script setup lang="ts">
import { ref } from 'vue'
const name = ref('Ada')
</script>

<template>
  <p>Hello, {{ name }}!</p>
  <p>{{ name.toUpperCase() }}</p>
</template>
```

Expressions work (one per interpolation); statements do not (`{{ if (ok) ... }}` is invalid).

### The Core Directives

```vue
<template>
  <!-- v-if / v-else — conditional rendering (removed from DOM) -->
  <p v-if="user">{{ user.email }}</p>
  <p v-else>Not signed in</p>

  <!-- v-show — toggles display only, element stays in DOM -->
  <p v-show="errors.length">{{ errors[0] }}</p>

  <!-- v-for — lists; :key tracks identity -->
  <li v-for="task in tasks" :key="task.id">
    {{ task.title }}
  </li>

  <!-- v-bind (: shorthand) — attribute binding -->
  <img :src="user.avatarUrl" :alt="user.name" />

  <!-- v-on (@ shorthand) — event listeners -->
  <button @click="save">Save</button>
  <form @submit.prevent="onSubmit">

  <!-- v-model — two-way form binding -->
  <input v-model="email" type="email" />
</template>
```

### Binding Arguments and Modifiers

Directives take arguments (`:href`, `@click`) and modifiers (`.prevent`, `.once`, `.trim`) that encode common behavior declaratively — `@submit.prevent="onSubmit"` replaces `event.preventDefault()` boilerplate.

### Class and Style Are Special

Vue merges bound classes with static ones:

```vue
<div class="card" :class="{ done: task.completed }">...</div>
```

### `computed` Over Method Calls

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
const tasks = ref<Task[]>([])
const doneCount = computed(() => tasks.value.filter(t => t.completed).length)
</script>

<template>
  <p>{{ doneCount }} done</p>
</template>
```

`computed` caches by dependencies; a method call re-executes on every render.

## Common Mistakes

- **Index as `:key` on mutable lists.** Reordering then reuses DOM for different items. Use a stable `id`.
- **`v-if` and `v-for` on the same element.** `v-if` evaluates first and cannot see the loop variable. Wrap the `v-for` in a `<template>` instead.
- **Mutating props in the template expression.** Bind and emit; never assign.
