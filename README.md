# stack-workflow

A Claude Code plugin that acts as a **senior project manager** and a **senior
full-stack developer** for JavaScript business systems.

It takes a project from a vague client conversation to a deployed app, in
stages, with one approval gate before any code is written — and it is built to
be cheap to run.

---

## Why this exists

Freelance software projects fail in two places, and neither of them is code:

1. **Building before the scope and price are agreed in writing.**
2. **Absorbing extra work for free because nobody wrote down what was excluded.**

This plugin puts a gate between "we talked about it" and "I started building",
and it produces the documents that gate needs: a spec with numbered acceptance
criteria, an honest estimate, and a written out-of-scope list.

---

## The workflow

```
intake ─▶ spec ─▶ estimate ─▶ ⛔ CLIENT APPROVAL ⛔ ─▶ plan ─▶ scaffold
                                                            │
                                        ┌───────────────────┘
                                        ▼
                              feature ─▶ feature ─▶ … ─▶ review ─▶ ship
```

| Command | What it does | Writes code? |
|---|---|---|
| `/stack:intake <name>` | Asks the questions a client never answers on their own | no |
| `/stack:spec <name>` | REQ / AC, role matrix, data model, state machine, API, screens, **out of scope** | no |
| `/stack:estimate <name>` | Days per feature, learning tax, price tiers, payment terms, client proposal | no |
| `/stack:plan <name>` | Build order and a file-by-file block per feature | no |
| `/stack:scaffold` | The monorepo: auth, roles, error handling, Docker, a working example feature | yes |
| `/stack:feature <name>` | One complete vertical slice | yes |
| `/stack:review [target]` | Parallel security + senior review, findings verified before reporting | no |
| `/stack:ship [domain]` | Production Docker, nginx, backups, deploy runbook, client handover doc | yes |
| `/stack:status` | Stage, approval, features done and left, next command | no |
| `/stack:explain <topic>` | One JS/Node concept explained to a Laravel developer | no |

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
concurrent users, `refetchInterval` on TanStack Query is the correct answer, not
a WebSocket.

---

## Cost and speed

The plugin is designed so a full project does not burn the budget on
boilerplate.

**Decide with the strong model. Write files with a cheap model in a subagent.
Never let generated code enter the main context.**

| Stage | Model | Effort | Why |
|---|---|---|---|
| spec, plan | inherit | high | Every later cost is decided here. Do not economise. |
| scaffold | haiku ×4 parallel | low | Pure boilerplate from a fixed template |
| feature | sonnet ×1 | medium | Judgement, but the pattern is already set |
| review | sonnet + inherit, parallel | high | Read-only, returns a short findings list |
| ship | haiku ×1 | low | Dockerfile, nginx, scripts — fully mechanical |

Three rules do more for cost than model choice:

- **Subagents return a file list, never file contents.**
- **Never read a file you just wrote.** Grep for the symbol, read 30 lines, edit.
- **One feature per command run.** Context growth makes every later token cost more.

Speed comes from running independent agents in one message: four during
`scaffold`, both reviewers during `review`.

**Where it deliberately spends more:** anything touching money, permissions,
authentication, or the core state machine is written by the strong model in the
main thread. Saving twenty cents on an authorization check is not a saving.

---

## For Laravel developers

The plugin carries a Laravel-to-Node translation guide and uses it when
explaining anything.

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
| Pint | Prettier + ESLint |
| PHPUnit | Vitest + Supertest |

It also warns about the traps that actually cost hours — `findUnique` returning
`null` instead of throwing, no lazy loading, a forgotten `await`, and Express not
catching errors from async handlers.

---

## Install

```bash
/plugin marketplace add Mohammad-Hasan-it-96/claude-stack-workflow
/plugin install stack@stack-workflow
```

Then, in a new project folder:

```
/stack:intake my-project
```

---

## What it creates in your project

```
_project/
  state.md            stage, approval, features done and left  ← single source of truth
  intake.md           raw requirements, assumptions, open questions
  spec.md             REQ / AC, roles, data model, state machine, API, out of scope
  estimate.md         internal: days, learning tax, risks
  proposal.md         client-facing: phases, price, terms
  plan.md             build order, one block per feature
  review-1.md         verified findings
  change-requests.md  scope added after approval, priced
  deploy.md           runbook
  handover.md         for the client, in plain language
```

`_project/state.md` front-matter owns the project stage. The stage is never
inferred from which files exist.

---

## Design decisions

- **One gate, not many.** `plan`, `scaffold`, and `feature` refuse to run while
  `approved_by_client: false`. Everything else is ungated. One gate that holds
  beats six that get skipped.
- **Non-mutating stages really are non-mutating.** `intake`, `spec`, `estimate`,
  `plan`, and `review` never touch source code.
- **Findings are verified before they are reported.** A review agent's finding is
  a claim. The command checks the file before showing it. A false finding costs
  more time than a missed one.
- **Scope changes get priced, not absorbed.** Anything outside `spec.md` goes to
  `change-requests.md` with a day estimate before code is written.
- **Vertical slices only.** A feature is model → migration → schema → service →
  controller → route → hook → page. Never all the models first.

---

## Status

Version 0.1.0 — early. Built while specifying a real taxi-office dispatch system.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
