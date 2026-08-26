# stack-workflow

A Claude Code plugin that acts as a **senior project manager** and a **senior
full-stack developer** for JavaScript business systems.

Client conversation → working spec → deployed app. **One project file. Eight
commands. No ceremony.**

---

## The idea

Most engineering workflows fail the same way: they produce documents nobody
reads twice. An intake file, a spec file, a plan file, a research file, ADRs, a
state file — each costs tokens to write, costs more to keep true, and gets
opened once.

This plugin writes **one** file: `PROJECT.md`. Requirements, roles, data model,
state machine, build order, decisions, and current state live in it together.
Its front-matter *is* the state.

Two more files are generated **only on demand**, because a real person outside
the project reads them:

| File | Reader | Command |
|---|---|---|
| `proposal.md` | the client, before signing | `/stack:proposal` |
| `handover.md` | the client, after delivery | `/stack:ship` |

Nothing else touches disk. Estimates, review findings, and status go in the
chat, where you actually read them.

---

## The flow

```
scope ─────────────▶ build ──────────────────────▶ ship
/stack:scope         /stack:scaffold               /stack:ship
/stack:proposal      /stack:feature  (repeat)
                     /stack:review
```

Three phases. `stage:` in the front-matter is one of three words. That is the
whole state machine — no blocker ids, no gate sequence, no history table.

| Command | What it does | Writes |
|---|---|---|
| `/stack:scope <name>` | One question round → requirements, roles, data model, states, build order. Days and price in the chat. | `PROJECT.md` |
| `/stack:proposal [lang]` | Client document: phases, prices, terms, out of scope | `proposal.md` |
| `/stack:scaffold` | The monorepo: auth, roles, error handling, Docker, one working example feature | code |
| `/stack:feature <name>` | One complete vertical slice | code |
| `/stack:review [target]` | Parallel security + senior review, findings verified before reporting | nothing |
| `/stack:ship [domain]` | Production Docker, nginx, backups, deploy section in README, client handover | code + `handover.md` |
| `/stack:status` | Phase, approval, features done and left, next command | nothing |
| `/stack:explain <topic>` | One JS/Node concept explained to a Laravel developer | nothing |

**`/stack:scope` replaces four separate steps** — intake, spec, estimate, and
plan — in one pass, because they were always one conversation pretending to be
four.

---

## It scales to the project

| Size | Signal | What scope does |
|---|---|---|
| small | under 5 features, one role, no money | 3 questions, ~20 lines, build the same session |
| normal | 5–15 features, 2–4 roles, money | 8 questions in one group, full `PROJECT.md` |
| large | 15+ features, or an external integration | same, plus integration risks in Decisions |

Guessing wrong toward *less* process is cheaper than guessing wrong toward more.

Questions come in **one** `AskUserQuestion` call. Two rounds is one round too
many. Anything that can be assumed gets written down as an assumption instead of
asked.

---

## The stack

Opinionated on purpose. One stack learned deeply beats five learned shallowly.

| Layer | Choice |
|---|---|
| Language | TypeScript, both sides |
| API | Express 4 + Prisma + PostgreSQL |
| Validation | Zod, shared between server and client |
| Auth | JWT access + revocable refresh token |
| Web | React 18 + Vite + TanStack Query + React Hook Form + Tailwind |
| Tests | Vitest + Supertest |
| Packaging | Docker Compose |

**Rejected on purpose:** NestJS (hides how Node works while you are learning
it), Next.js (SSR and server components buy nothing for a dashboard behind a
login), MongoDB (business systems need relations and transactions), tRPC (breaks
when a mobile app needs the same API).

Realtime is added **only** when a requirement needs it. For a dashboard under 50
concurrent users, `refetchInterval` is the correct answer, not a WebSocket.

---

## Cost and speed

**Decide with the strong model. Write files with a cheap model in a subagent.
Never let generated code enter the main context.**

| Command | Model | Effort |
|---|---|---|
| `/stack:scope` | inherit | high — every later cost is decided here |
| `/stack:scaffold` | haiku ×4 parallel | low |
| `/stack:feature` | sonnet ×1 | medium |
| `/stack:review` | sonnet + inherit, parallel | high |
| `/stack:ship` | haiku ×1 | low |

Four rules do more for cost than model choice:

- **Subagents return a file list, never file contents.**
- **Never read a file you just wrote.** Grep the symbol, read 30 lines, edit.
- **One feature per command run.** Context growth makes every later token dearer.
- **One project file.** Nothing to re-read, nothing to keep in sync.

Speed comes from running independent agents in one message: four during
`scaffold`, both reviewers during `review`.

**Where it deliberately spends more:** anything touching money, permissions,
authentication, or the core state machine is written by the strong model in the
main thread. Saving twenty cents on an authorization check is not a saving.

---

## For Laravel developers

The plugin carries a Laravel-to-Node translation guide and uses it whenever it
explains anything.

| Laravel | This stack |
|---|---|
| Routes + Controllers | Express Router + controller functions |
| Eloquent + Migrations | Prisma Client + `prisma migrate` |
| Form Request | Zod schema (validates *and* types) |
| Middleware | Express middleware |
| Service Container | plain `import` — there is no container |
| Blade / Inertia | React + Vite |
| Sanctum | JWT + refresh token, ~80 lines you own |
| `artisan make:model -mcr` | `/stack:feature <name>` |
| PHPUnit | Vitest + Supertest |

It also warns about the traps that actually cost hours — `findUnique` returning
`null` instead of throwing, no lazy loading, a forgotten `await`, and Express
not catching errors thrown inside async handlers.

`/stack:explain <topic>` answers in four parts: the Laravel equivalent, the
difference that matters, a real example from your codebase, and the trap. It is
told you are a senior developer, so it never explains what a function is.

---

## Install

```bash
/plugin marketplace add Mohammad-Hasan-it-96/claude-stack-workflow
/plugin install stack@stack-workflow
```

Then, in a new project folder:

```
/stack:scope my-project
```

---

## What ships in the plugin

```
plugins/stack/
  commands/    8 commands
  agents/      5 agents, each pinned to a model tier
  rules/       stack · conventions · flow · model-policy · laravel-map
  skills/      stack-conventions (auto-loads while writing code)
  templates/   PROJECT.md
```

---

## Design decisions

- **One file, not eight.** A document with no second reader is waste. Estimates
  and review findings go in the chat instead.
- **A reminder, not a gate.** If the client has not approved, `/stack:scaffold`
  asks once and accepts either answer. The user runs their own business.
- **Findings are verified before they are reported.** A review agent's finding
  is a claim; the command checks the file before showing it. A false finding
  costs more time than a missed one.
- **Scope changes get one table row and a sentence** — not a change-request
  document, and not silence.
- **Vertical slices only.** model → migration → schema → service → controller →
  route → hook → page. Never all the models first.

---

## Status

Version 0.2.0. Built while specifying a real taxi-office dispatch system.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
