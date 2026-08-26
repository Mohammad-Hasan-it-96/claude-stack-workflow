# The Stack (binding)

This plugin is opinionated on purpose. One stack, learned deeply, beats five
stacks learned shallowly. Do not introduce an alternative unless the user
explicitly asks.

## Chosen stack

| Layer | Choice | Why this one |
|---|---|---|
| Language | TypeScript (both sides) | Types catch mistakes a beginner cannot see yet. Prisma and Zod give real autocomplete. |
| Runtime | Node 20 LTS or newer | Native fetch, stable ESM, long support. |
| HTTP server | Express 4 | Nothing is hidden. Every request path is code you can read. |
| ORM | Prisma | Closest thing to Eloquent plus migrations. One schema file. |
| Database | PostgreSQL 16 | Real constraints, real transactions, JSON when needed. |
| Validation | Zod | One schema validates the request AND types the TypeScript. |
| Auth | JWT access token + refresh token | No session store needed. Works for web and mobile clients. |
| Frontend build | Vite | Fast, simple, no framework conventions to learn. |
| UI | React 18 + TypeScript | The thing being learned. |
| Server state | TanStack Query | Handles caching, refetch, loading and error states. |
| Forms | React Hook Form + the same Zod schema | Client and server validate identically. |
| Styling | Tailwind CSS | No CSS naming decisions. Fast for dashboards. |
| Routing | React Router v6 | Plain, explicit routes. |
| Tests | Vitest (unit), Supertest (API), Playwright (critical flows only) | |
| Realtime | Socket.IO, only when a requirement needs it | |
| Background jobs | BullMQ + Redis, only when a requirement needs it | |
| Packaging | Docker Compose | One command runs api + web + db locally and on a VPS. |

## Rejected on purpose - do not suggest these

| Rejected | Reason |
|---|---|
| NestJS | Decorators and dependency injection hide how Node works. The user is learning Node, not learning Nest. Revisit only for a team of 5 or more developers. |
| Next.js | These are dashboards behind a login. No SEO need. SSR and server components are three new concepts for zero benefit. |
| Mongoose / MongoDB | Business systems need relations, joins, and transactions. Postgres is the right default. |
| Redux or Zustand as the first choice | TanStack Query removes most global state. Add a client store only for state that is genuinely not server data. |
| tRPC | Ties the client to the server. Breaks the moment a mobile app needs the same API. |
| Turborepo / Nx at the start | npm workspaces is enough until the build is actually slow. |

## When to break the stack

Only these reasons are valid, and each must be written into the PROJECT.md Decisions table:

1. A hard client requirement the stack genuinely cannot meet.
2. The client already runs infrastructure that forces a choice (for example, they only have MySQL).
3. The user explicitly asks for a different tool.

"It would be more modern" is not a reason. "It would be faster to type" is not a
reason.

## Realtime decision rule

Add Socket.IO only if a requirement says data must reach a screen without the
user acting. Everything else is solved by TanStack Query with `refetchInterval`.

Polling every 10 seconds is not a hack. For a dashboard with under 50 concurrent
users it is the correct engineering answer, and it costs zero extra
infrastructure. Say this out loud when the user asks for "realtime" - most of the
time they mean "the screen should update by itself", and polling does that.
