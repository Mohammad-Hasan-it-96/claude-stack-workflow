# Laravel to Node translation

The user of this plugin is an experienced Laravel developer learning the
JavaScript stack. When you explain anything, anchor it to the Laravel equivalent
first, then show the JavaScript. Never explain a JavaScript concept as if the
user were new to programming - they are not. They are new to this ecosystem.

## Concept map

| Laravel | This stack | Important difference |
|---|---|---|
| `routes/api.php` | `src/modules/<domain>/<domain>.routes.ts` | Routes are split per module, not one big file. |
| Controller | Controller function | No base class, no auto-injection. Plain function `(req, res, next)`. |
| Service class | Service module | Plain exported functions. No container, no binding, no interface. Just `import`. |
| Form Request | Zod schema | Validates AND produces the TypeScript type. One object does both jobs. |
| Middleware | Express middleware | Order matters a lot. Registered in `app.ts` top to bottom. |
| Eloquent Model | Prisma model in `schema.prisma` | Prisma has no model class and no methods on rows. Rows are plain objects. Logic lives in services. |
| `Model::find()` | `prisma.model.findUnique()` | Returns `null`, does not throw. `findUniqueOrThrow` throws. |
| Eloquent relationships | Prisma `include` / `select` | Nothing is lazy-loaded. If you did not `include` it, it is not there. There is no N+1 by accident, but also no magic. |
| `php artisan migrate` | `npx prisma migrate dev` | Migrations are generated from the schema file diff, not written by hand. |
| Seeder | `prisma/seed.ts` | Plain script. |
| `.env` + `config()` | `.env` + a typed `lib/config.ts` | Parse and validate env with Zod at boot. Crash on start if a var is missing, never at 2am. |
| Service Container / DI | plain `import` | There is no container. This is simpler, not worse. |
| Service Provider | `app.ts` wiring | Everything is registered explicitly in one readable file. |
| Facade (`Cache::`, `DB::`) | imported singleton (`prisma`, `redis`) | Exported once from `lib/`, imported where needed. |
| Sanctum / Passport | JWT access + refresh token | You implement it. It is about 80 lines. Do not add a library that hides it. |
| Gate / Policy | a `can()` helper in the service | Check ownership inside the service, not in the controller. |
| Queue + Job | BullMQ + Redis | Only add when a requirement needs it. |
| Scheduler (`schedule:run`) | node-cron or a system cron calling a script | |
| Blade / Inertia | React + Vite | Full page/state separation. The server sends JSON only. |
| `php artisan serve` | `npm run dev` | Two dev servers: api on 3000, web on 5173. Vite proxies `/api` to the api. |
| `artisan make:model -mcr` | `/stack:feature <name>` | This plugin's command generates the vertical slice. |
| Tinker | `npx prisma studio` for data; a `scripts/` file for logic | |
| Pint | Prettier + ESLint | |
| PHPUnit | Vitest + Supertest | |
| Composer | npm workspaces | |

## Traps that catch Laravel developers

Say these out loud when they become relevant. They are the actual sources of
lost hours, not syntax.

1. **`null` versus exception.** Eloquent's `findOrFail` throws. Prisma's
   `findUnique` returns `null` silently. Forgetting to check `null` is the single
   most common bug when moving over.

2. **No lazy loading.** `trip.driver` is `undefined` unless you wrote
   `include: { driver: true }`. In Laravel it would silently run a second query.
   Here it is just missing. This is better, but it surprises people.

3. **`async` / `await` everywhere.** Almost every database call returns a
   Promise. A forgotten `await` gives you a `Promise` object where you expected
   data, and the error appears far away from the cause.

4. **No global error handler by default.** Laravel catches exceptions for you.
   In Express, an error thrown inside an `async` handler is *not* caught unless
   you wrap it. Use an `asyncHandler` wrapper or Express 5. This is mandatory,
   not optional.

5. **`this` and arrow functions.** Losing `this` when passing a method as a
   callback is a real JavaScript problem with no PHP equivalent. Prefer plain
   exported functions over classes and the problem disappears.

6. **Dates.** JavaScript `Date` is weak. Use `date-fns` for formatting and
   arithmetic. Never do date math with raw milliseconds.

7. **Numbers.** There is one number type and it is a float. `0.1 + 0.2` is not
   `0.3`. This is why money is stored as an integer.

8. **Timezone in the browser.** The server sends UTC. The browser renders in the
   viewer's local timezone by default, which may not be the business timezone.
   Decide once and be explicit.

9. **The frontend has no session.** There is no `auth()` helper available
   everywhere. The token lives in memory or storage and is attached by an axios
   interceptor. State is a thing you now manage on purpose.

10. **Two servers, one origin problem.** CORS errors in development are normal
    and are solved by the Vite proxy, not by setting `Access-Control-Allow-Origin: *`.

## How to teach during work

When a command in this plugin writes code that uses a pattern the user has not
met yet, add a short note in the final message - not in the code comments:

> Note: `asyncHandler` wraps the controller so a thrown error reaches the error
> middleware. Laravel does this for you; Express does not.

Keep it to two or three sentences. One concept per note. Do not turn a build
into a lecture.
