# Workshop: TaskFlow — Fullstack Task Manager

## Objective

Build and ship **TaskFlow**, a fullstack task management app: Vue 3 frontend, Elysia API, MongoDB persistence, JWT auth — running on Bun, tested, and deployable with Docker.

This workshop uses everything from the path. There is no starter code; every decision is yours.

## Product Requirements

### Authentication

- Sign up with email + name + password (bcrypt-hashed, never stored plaintext)
- Login returns a JWT (1h expiry) and the current user
- Logout clears the client session
- All task routes require a valid token; users see only their own data

### Tasks

- Create a task: `title` (1–200 chars), `priority` (`low` | `high`), optional `dueDate`
- List my tasks, newest first, filterable by `?priority=`
- Complete / reopen a task
- Delete a task
- Optional stretch: tags array

### UI (Vue 3 + Router + Pinia)

- `/login` — login + signup forms with client validation (Zod), field-level errors
- `/` — task list: add form, filter tabs, complete/delete actions, empty and loading states
- Router guard redirects unauthenticated users to `/login` with a `redirect` query, and sends them back after login
- Auth state lives in a Pinia store; token persisted to `localStorage`

## Data Model

Design the schema yourself. Minimum shape:

```
users   { email (unique), passwordHash, name }
tasks   { title, priority, dueDate, completed, user (ref → users), timestamps }
```

Justify one embed-vs-reference decision in your README (e.g., why `user` is a reference, not embedded).

## API Contract

Define with `t.Schema` — validation and response types are required:

| Method | Path          | Auth | Notes |
|--------|---------------|------|-------|
| POST   | /auth/signup  | no   | 201 + token |
| POST   | /auth/login   | no   | 200 + token, or 401 |
| GET    | /tasks        | yes  | mine only, `?priority=` |
| POST   | /tasks        | yes  | 201, 422 on invalid |
| PATCH  | /tasks/:id    | yes  | complete/reopen; 404 if not mine |
| DELETE | /tasks/:id    | yes  | 404 if not mine |

Error responses share one shape: `{ error: { code, message } }`.

## Architecture Expectations

- `server/` — Elysia app as route modules + plugins (auth `derive`, repos via `decorate`), MongoDB singleton in `db.ts`, config validated at startup in `env.ts`
- `frontend/` — Treaty client typed against `type App`, composables per domain (`useTasks`), components small and prop-driven
- CORS via Vite dev proxy in development; nginx `/api` proxy in production
- Tests: `bun test` API suite covering validation, auth (401), and ownership (user A cannot read user B's task — expect 404)

## Milestones

1. **Skeleton** — Bun projects for `server/` and `frontend/`; Mongo via Compose; `GET /tasks` returns a fixture → verify: both dev servers running, one endpoint round-trip
2. **Auth** — signup/login routes, JWT plugin, Pinia store, router guard → verify: signup → refresh page → still signed in; task routes 401 without token
3. **Tasks CRUD** — schema, repository, routes, list UI with filters → verify: create/complete/delete in the UI, data survives server restart
4. **Hardening** — error contract in `onError`, form validation both sides, loading/empty/error UI states → verify: invalid input shows field errors; API returns clean 422s
5. **Testing** — API test suite with a fresh database per run → verify: `bun test` green, including the ownership test
6. **Ship** — Dockerfiles for both apps, nginx config with `/api` proxy and history fallback, root Compose file → verify: `docker compose up --build` gives a working app on `:80`

## Deliverables

- Repo (or monorepo) with `server/` + `frontend/`, both starting with one command each
- Passing tests (`bun test`, component tests optional)
- `docker compose up --build` serving the full app
- README: setup, env vars (`.env.example` committed), your embed-vs-reference justification, what you would build next

## Definition of Done

- [ ] Two users, isolated data — proven by test, not by hope
- [ ] No secret in code; no `any` on the API boundary
- [ ] Frontend compiles against server types — rename a field, watch it break at compile time
- [ ] Fresh clone → `docker compose up --build` → working app
