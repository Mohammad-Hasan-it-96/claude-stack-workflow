---
description: Create the monorepo skeleton - Express + Prisma API, React + Vite web app, shared Zod package, auth, roles, error handling, Docker Compose. Fans out to four cheap parallel codegen agents. Run once per project.
argument-hint: [project-name]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task
---

# /stack:scaffold

Build the project skeleton. Run once. It produces a running app with working
login before any business feature exists.

Read first:
- `${CLAUDE_PLUGIN_ROOT}/rules/stack.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/conventions.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/model-policy.md`

## Gate

If `PROJECT.md` has `approved: false`, say **one line**:

> Client has not approved in writing yet. Continue anyway?

Accept either answer and act on it. Never raise it again in this project. It is
a reminder, not a gate.

## How this command spends tokens

You make the decisions in this thread. You do **not** write the boilerplate
yourself. Four `codegen` agents on the cheap model write it in parallel. Their
output is a file list, not code, so generated files never enter this context.

## Step 1 - decide, in this thread, before spawning anything

Write these decisions down in your prompt to the agents. They will not ask.

- Project name, database name, ports
- Roles enum (from the PROJECT.md role matrix)
- Is the UI Arabic / RTL?
- The one seeded admin user
- Which example feature the users module demonstrates

## Step 2 - spawn four codegen agents in ONE message

They must not share a file. This split is chosen so they do not:

**Agent A - root and shared**
```
package.json (npm workspaces: apps/*, packages/*)
tsconfig.base.json
.env.example
.gitignore
docker-compose.yml        postgres + api + web
README.md                 how to run
packages/shared/package.json
packages/shared/src/index.ts
packages/shared/src/schemas/auth.ts
packages/shared/src/schemas/common.ts   pagination query + meta
```

**Agent B - API core**
```
apps/api/package.json, tsconfig.json
apps/api/prisma/schema.prisma       User, Role enum, RefreshToken
apps/api/prisma/seed.ts             one admin
apps/api/src/lib/config.ts          env parsed with Zod, throws at boot
apps/api/src/lib/prisma.ts          one PrismaClient
apps/api/src/lib/errors.ts          AppError + error codes
apps/api/src/lib/async-handler.ts
apps/api/src/middleware/error.ts    last middleware, AppError to JSON
apps/api/src/middleware/auth.ts     requireAuth, requireRole
apps/api/src/app.ts                 helmet, cors, json, mount routers, error mw
apps/api/src/server.ts              listen only
```

**Agent C - API auth and users modules**
```
apps/api/src/lib/jwt.ts
apps/api/src/modules/auth/*         login, refresh, logout, me
apps/api/src/modules/users/*        list (paginated), create, deactivate
apps/api/tests/auth.test.ts
apps/api/tests/users.test.ts
```
Tell Agent C the exact export names Agent B creates (`AppError`, `asyncHandler`,
`requireAuth`, `requireRole`, `prisma`) so it imports them without looking.

**Agent D - web app**
```
apps/web/package.json, tsconfig.json, vite.config.ts (proxy /api -> :3000)
apps/web/tailwind.config.js, index.html, src/main.tsx
apps/web/src/lib/api.ts             axios + token interceptor + 401 refresh once
apps/web/src/lib/auth.tsx           AuthProvider, useAuth
apps/web/src/components/ui/         Button, Input, Select, Table, Modal, Toast
apps/web/src/components/AppShell.tsx
apps/web/src/features/auth/LoginPage.tsx
apps/web/src/features/users/        api.ts, UsersListPage, UserFormPage
apps/web/src/routes.tsx
```

## Step 3 - wire and verify, in this thread

The agents cannot run the app. You do.

```
npm install
docker compose up -d db
npx prisma migrate dev --name init
npx prisma db seed
npx tsc --noEmit
npm run dev
```

Fix failures yourself with targeted edits - Grep for the symbol, read 30 lines
around it, edit. Do not read whole generated files. Do not re-run a codegen
agent for a two-line fix.

Do not report success until `npm run dev` serves both apps and you can log in as
the seeded admin.

## Non-negotiable in the generated code

1. **Config fails fast.** `lib/config.ts` parses `process.env` with Zod and
   throws at boot. `process.env` is read nowhere else.
2. **`app.ts` never calls `listen`.** `server.ts` does. This is what makes
   Supertest work.
3. **Every async controller is wrapped in `asyncHandler`.** Express 4 does not
   catch a rejected promise. This is the single most common production bug for a
   Laravel developer moving to Express.
4. **Auth is real.** bcrypt cost 12, access token 15 minutes, refresh token
   7 days stored in the database so it can be revoked, and a web interceptor
   that refreshes once on 401 then retries.
5. **RTL from the start if the project is Arabic.** `dir="rtl"` on `<html>`,
   logical Tailwind classes (`ms-`, `me-`, `ps-`, `pe-`), an Arabic-capable
   font. Retrofitting this later means touching every screen.
6. **The users module actually works** - list with pagination, create,
   deactivate, empty state. It is the pattern every later feature copies.

## Step 4 - close out

Set `stage: build` in `PROJECT.md`.

End your message with two or three sentences about **one** pattern that has no
Laravel equivalent - `asyncHandler`, the axios interceptor, or npm workspaces.
One concept only. Do not turn a build into a lecture.

## Next step

Run `/stack:feature <first feature with status: todo>`.
