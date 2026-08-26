---
description: Turn intake notes into a real specification - numbered requirements, acceptance criteria, data model, API surface, and screen list. Writes _project/spec.md. Writes no code.
argument-hint: <project-name>
allowed-tools: Read, Write, Edit, Glob, Grep, Task
---

# /stack:spec

Act as a **senior product manager working with a senior architect**. Turn
`_project/intake.md` into a specification that a developer could build from
without asking the client another question.

Read first:
- `${CLAUDE_PLUGIN_ROOT}/rules/lifecycle.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/stack.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/conventions.md`

This stage writes nothing outside `_project/`.

## Steps

1. Confirm `_project/state.md` has `stage: spec`. If `intake.md` is missing,
   stop and tell the user to run `/stack:intake` first.

2. Resolve every `OQ-n` from intake. Any that is still open must either be
   answered now or converted into a written assumption. Do not carry an open
   question into the spec.

3. Write `_project/spec.md` from `${CLAUDE_PLUGIN_ROOT}/templates/spec.md`.

4. Set `stage: estimate` in `_project/state.md`.

## What the spec must contain

**Requirements** - `REQ-1`, `REQ-2`, ... One sentence each, in business
language. If a requirement needs "and" twice, split it.

**Acceptance criteria** - `AC-1.1` under each `REQ`. Written as something you
can check by clicking:

> AC-3.2: When the operator assigns a driver, the trip status becomes
> "assigned" and the customer app shows the driver name and phone.

**Roles and permission matrix** - a table of role by action with yes/no. This
table becomes the authorization code later, so be exact.

**Data model** - the Prisma models, their fields with types, and the relations.
Include:
- every status enum with all its values
- which fields are nullable and why
- money as integer in the smallest unit
- `createdAt` / `updatedAt` on everything
- soft delete (`deletedAt`) on anything a user can "delete"

**State machine** - for the core object, list every allowed transition:
`new -> assigned -> in_progress -> completed -> paid`, plus `cancelled` from
which states, and who is allowed to make each move.

**API surface** - a table of method, path, purpose, role required. Follow
`/api/v1/` and the response shapes in `conventions.md`.

**Screens** - a list per role. For each: what it shows, what actions it has,
and the empty state.

**Out of scope** - an explicit list. This is the section that protects you.
Anything discussed but not being built goes here by name.

**Non-functional** - language and direction, timezone, hosting, browser
support, expected volume.

## Quality bar

Before finishing, check each item and fix what fails:

- Every `AC` is checkable by a person clicking, not by reading code.
- Every screen maps to at least one `REQ`.
- Every `REQ` maps to at least one screen or one API endpoint.
- Every status in the state machine appears in the data model enum.
- No requirement says "etc.", "and so on", or "similar".

## Optional deeper pass

If the project is large or the money matters, spawn one `api-designer` agent to
turn the data model into the endpoint table and the Zod schema list, and review
its output yourself before writing it in. Do not spawn agents for a small
project - it is slower than doing it directly.

## Next step

Run `/stack:estimate $1`.
