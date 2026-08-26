---
name: stack-conventions
description: Conventions for writing code in an Express + Prisma + React + Vite TypeScript monorepo built by the stack workflow. Load before writing or editing any file under apps/api, apps/web, or packages/shared, and whenever adding an endpoint, a Prisma model, a Zod schema, a service, a controller, or a React page in such a project. Also load when a Laravel developer asks how something in this stack maps to Laravel.
---

# Stack conventions

You are working in a monorepo built by the `stack` workflow plugin. The stack is
fixed, the layers are strict, and the shape is already established by the
`users` module. Copy it rather than inventing.

## Before writing code

Read these, once per session:

- `${CLAUDE_PLUGIN_ROOT}/rules/conventions.md` - layers, naming, response shapes
- `${CLAUDE_PLUGIN_ROOT}/rules/stack.md` - what is allowed and what is rejected

If the user is a Laravel developer, also keep
`${CLAUDE_PLUGIN_ROOT}/rules/laravel-map.md` in mind for explanations.

## The five rules people break

1. **A controller never touches `prisma`.** Parse with Zod, call one service
   function, shape the response. If it is longer than 15 lines, logic leaked in.

2. **A service never sees `req` or `res`.** Plain arguments in, plain data out,
   or it throws an `AppError`.

3. **A React component never calls `axios`.** It calls a hook from
   `features/<domain>/api.ts`.

4. **Every async controller is wrapped in `asyncHandler`.** Express 4 does not
   catch a rejected promise. A missing wrapper means the request hangs and the
   error never reaches the error middleware.

5. **A Zod schema lives once, in `packages/shared`.** The API and the form both
   import it. A second copy will drift.

## Vertical slices only

Add a feature in this order, all the way through, before starting the next:

Prisma model -> migration -> shared Zod schema -> service -> controller -> route
-> register in `app.ts` -> API hook -> page -> register in `routes.tsx`

Never write all the models first. A half-built horizontal layer hides
integration bugs until the end.

## Always

- Paginate every list endpoint. Default 20, max 100.
- Check ownership in the service, not just authentication in the middleware.
- Money as an integer in the smallest unit. Never a float.
- Store UTC; convert in the UI.
- Loading, empty, and error state on every page.
- Named error codes in SCREAMING_SNAKE_CASE, stable once shipped.
- Arabic projects: `dir="rtl"` and logical Tailwind classes (`ms-`, `me-`,
  `ps-`, `pe-`), never `ml-` / `mr-`.

## Never

- `any`. Use `unknown` and narrow.
- `$queryRaw` with string interpolation.
- `req.body` passed straight into a Prisma write.
- `process.env` read outside `lib/config.ts`.
- A new library that `rules/stack.md` rejects.
- An abstraction with one caller.

## Verify before saying done

```
npx tsc --noEmit
npm run test -w apps/api
```

Then use the feature once in the browser. Code that compiles but cannot be used
is not finished.
