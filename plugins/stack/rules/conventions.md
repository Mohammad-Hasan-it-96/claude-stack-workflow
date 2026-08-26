# Code conventions (binding)

## Repository shape

```
<project>/
  apps/
    api/
      src/
        modules/<domain>/          one folder per business domain
          <domain>.routes.ts       Express router, thin
          <domain>.controller.ts   reads req, calls service, sends res
          <domain>.service.ts      ALL business logic lives here
        middleware/                auth, error handler, request logger
        lib/                       prisma client, jwt, config, logger
        app.ts                     builds the Express app (no listen)
        server.ts                  listen only - keeps app testable
      prisma/
        schema.prisma
        migrations/
        seed.ts
      tests/
    web/
      src/
        features/<domain>/         mirrors the api module names
          api.ts                   TanStack Query hooks for this domain
          <Domain>ListPage.tsx
          <Domain>FormPage.tsx
          components/
        components/ui/             shared dumb components
        lib/                       axios instance, auth, helpers
        routes.tsx
        main.tsx
  packages/
    shared/
      src/
        schemas/<domain>.ts        Zod schemas imported by BOTH sides
        types.ts
  docker-compose.yml
  .env.example
```

## The one rule above all: build vertical slices

A feature is a vertical slice: Prisma model, migration, Zod schema in
`packages/shared`, service, controller, route, API hook, page.

Never build all the models first, then all the controllers. Build one feature
end to end, see it work in the browser, then start the next one. A half-built
horizontal layer proves nothing and hides integration bugs until the end.

## Naming

| Thing | Style | Example |
|---|---|---|
| Files | kebab-case | trip-status.service.ts |
| React components | PascalCase file and export | TripListPage.tsx |
| Variables, functions | camelCase | assignDriver |
| Types, interfaces, Zod schemas | PascalCase | CreateTripInput |
| Database tables | snake_case plural, set with @@map | @@map("trip_statuses") |
| Prisma models | PascalCase singular | model Trip |
| API routes | kebab-case plural nouns | /api/v1/trips/:id/assign |
| Env vars | SCREAMING_SNAKE_CASE | DATABASE_URL |

## Layer rules - do not cross them

- **Routes** only wire a path to a controller and list middleware. No logic.
- **Controllers** only do three things: parse input with Zod, call one service
  function, shape the HTTP response. A controller must never touch `prisma`.
- **Services** hold all business logic and all database access. A service must
  never see `req` or `res`. It takes plain arguments, returns plain data, or
  throws a typed error.
- **React components** must never call `axios` directly. They call a hook from
  `features/<domain>/api.ts`.

If a controller is longer than about 15 lines, business logic has leaked into
it. Move it down to the service.

## Errors

Define one `AppError` class with `statusCode`, `code`, and `message`. Services
throw it. One error-handling middleware, registered last, turns it into JSON.
Never send a raw stack trace to the client in production.

```json
{ "error": { "code": "TRIP_ALREADY_ASSIGNED", "message": "..." } }
```

Error codes are SCREAMING_SNAKE_CASE and stable. The web app and any mobile app
branch on them, so renaming a code is a breaking change.

## API response shape

- List: `{ "data": [...], "meta": { "page": 1, "perPage": 20, "total": 97 } }`
- Single: `{ "data": { ... } }`
- Error: `{ "error": { "code": "...", "message": "...", "fields": { ... } } }`

Always paginate list endpoints. Default 20, maximum 100. A client whose table
grows to 5,000 rows must not become a rewrite.

Version the API from day one: `/api/v1/...`. It costs nothing now and saves a
painful migration later when a mobile app is already installed on phones.

## Validation

Every schema lives in `packages/shared/src/schemas/`, exactly once. The API
validates the request with it. The React form validates with the same object.
Two copies will drift - that is not a style opinion, it is a guarantee.

```ts
export const CreateTripInput = z.object({
  customerId: z.string().uuid(),
  pickupText: z.string().min(3).max(200),
})
export type CreateTripInput = z.infer<typeof CreateTripInput>
```

## Security floor - non-negotiable

- Never trust `req.body`. Zod-parse before use, every time.
- Never build SQL by string concatenation. Prisma only. Using `$queryRaw`
  requires an explicit note in the PROJECT.md Decisions table saying why.
- Hash passwords with bcrypt, cost 12.
- Keep the JWT secret in `.env`. Never commit `.env`. Always ship `.env.example`.
- Check ownership, not just login. "Is this user authenticated?" is a different
  question from "does this trip belong to this user's office?"
- Use helmet, cors with an explicit origin allowlist, and a rate limit on
  `/auth/*`.
- Never log a token, a password, or a request body that contains either.

## Money and time

- Money: store as an integer in the smallest unit (cents, piastres). Never use
  `float` or `Decimal` in JavaScript for arithmetic on money.
- Time: store UTC as `DateTime`. Convert in the UI only. Record the timezone the
  business runs in once, in config.

## Comments

Write a comment only where the reason is not obvious from the code. Do not
narrate what the next line does. Match the comment density of the surrounding
file.
