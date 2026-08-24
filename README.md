# Vue + Elysia Fullstack Developer

A directory-per-topic learning path for building fullstack web applications with Vue 3, Elysia, TypeScript, and MongoDB — all running on Bun.

## Learning Objectives

By the end of this path you can:

- Build Vue 3 frontends with the Composition API, Vue Router, and Pinia
- Create type-safe Elysia APIs with end-to-end types via Eden Treaty
- Model and query MongoDB data with Mongoose
- Ship a fullstack app with auth, testing, and Docker

## Prerequisites

- Basic JavaScript syntax
- Familiarity with HTML/CSS
- Completed the JS/TS foundations (Bun setup, TypeScript basics) in [nodejs-typescript-backend](https://github.com/TP-Coder-Innovation-Hub/nodejs-typescript-backend)

## Structure

```
README.md                              — This file
01-vue-fundamentals/                   — Components, templates, reactivity
  01-why-vue.md
  02-template-syntax.md
  03-reactivity-composables.md
  04-components-props-events.md
02-vue-app-building/                   — Router, state, forms
  01-vue-router.md
  02-pinia-state.md
  03-forms-validation.md
03-elysia-backend/                     — Beyond the first API
  01-elysia-lifecycle-plugins.md
  02-type-safety-t-schema.md
  03-error-handling.md
04-mongodb-bun/                        — Persistence with Mongoose
  01-connecting-mongodb-bun.md
  02-mongoose-schemas.md
  03-repository-pattern.md
05-fullstack-integration/              — Frontend meets backend
  01-eden-treaty-with-vue.md
  02-auth-flow-jwt.md
  03-cors-env-config.md
06-production/                         — Testing and deployment
  01-testing-bun-eden.md
  02-deployment-docker.md
99-workshop/                           — Full project
  01-workshop-spec.md
```

## The Stack

| Layer     | Technology                              |
|-----------|-----------------------------------------|
| Runtime   | Bun (package manager, test runner, dev) |
| Language  | TypeScript everywhere                   |
| Frontend  | Vue 3 Composition API + `<script setup>`|
| Backend   | Elysia                                  |
| Database  | MongoDB + Mongoose                      |
| Types     | Eden Treaty (end-to-end type safety)    |
