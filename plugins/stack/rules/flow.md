# Flow (binding)

## One file

The project has **one** working document: `PROJECT.md` in the repo root.

It holds the requirements, the data model, the build order, the decisions, and
the current state, together. Its front-matter is the state.

Do not create `intake.md`, `spec.md`, `plan.md`, `research.md`, `review-1.md`,
`change-requests.md`, or an ADR folder. A document nobody reads twice is a
document that cost tokens to write and will cost more to keep true.

Two files are generated **on demand only**, because a real person outside the
project reads them:

| File | Reader | Command |
|---|---|---|
| `proposal.md` | the client, before signing | `/stack:proposal` |
| `handover.md` | the client, after delivery | `/stack:ship` |

Nothing else gets written to disk.

## Three phases, not eight stages

```
scope  ─▶  build  ─▶  ship
```

| Phase | Commands | What it means |
|---|---|---|
| scope | `/stack:scope`, `/stack:proposal` | We know what and how much |
| build | `/stack:scaffold`, `/stack:feature`, `/stack:review` | Code exists and runs |
| ship | `/stack:ship` | It is on a server and the client has it |

`stage:` in the front-matter is one of these three words. That is the whole
state machine. There is no stage that must be entered before another, no
blocker id, and no history table.

## Scale the process to the project

Read the size before choosing how much ceremony to apply.

| Size | Signal | How to scope |
|---|---|---|
| small | under 5 features, one role, no money | Ask 3 questions. Write 20 lines. Start building the same session. |
| normal | 5-15 features, 2-4 roles, money involved | Ask 8 questions in one group. Write the full `PROJECT.md`. |
| large | 15+ features, or an external integration | Normal, plus name the integration risks in Decisions. |

A small project that gets the large treatment is the failure mode this plugin
exists to avoid. Guessing wrong toward *less* process is cheaper than guessing
wrong toward more.

## The one check

Before `/stack:scaffold` writes code, if `approved: false`, say **one line**:

> Client has not approved in writing yet. Continue anyway?

Accept either answer, act on it, and never raise it again in that project. It is
a reminder, not a gate. The user is an adult running their own business.

## Scope changes

When the client asks for something not in `PROJECT.md`:

1. Add a row to the `Changes after approval` table with a day estimate.
2. Say the cost in one sentence.
3. Build it.

Do not open a negotiation. Do not write a change-request document. One row.

## Ask questions in one group, once

Never ask a question you can answer from the code, the conventions, or a sane
default. When you must ask, put every question in one `AskUserQuestion` call and
carry on. Two rounds of questions is one round too many.

Write down what you assumed instead of asking about it. An assumption in
`PROJECT.md` that the user never corrects is a decision that cost nothing.
