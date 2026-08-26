---
description: Turn the approved spec into a file-by-file build plan and a feature build order. Writes _project/plan.md. Writes no source code. Blocked until the client has approved.
argument-hint: <project-name>
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /stack:plan

Act as a **senior full-stack developer** planning the build.

Read first:
- `${CLAUDE_PLUGIN_ROOT}/rules/lifecycle.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/stack.md`
- `${CLAUDE_PLUGIN_ROOT}/rules/conventions.md`

## Gate

If `_project/state.md` has `approved_by_client: false`, stop. Tell the user in
one sentence that planning before written approval is how unpaid work happens,
and ask them to confirm. If they confirm, continue and do not mention it again.

This stage writes nothing outside `_project/`.

## Steps

1. Read `spec.md` and `estimate.md`.
2. Write `_project/plan.md`.
3. Set `stage: scaffold` in `_project/state.md`.

## What the plan must contain

**Build order.** The list of features in the order they will be built, with a
one-line reason for the order. Rules for ordering:

- Auth and roles first. Everything else depends on knowing who is asking.
- Then the core object with its simplest possible flow, end to end.
- Then the states and transitions on that object.
- Then money and settlement.
- Then reports.
- Realtime, uploads, and integrations last - they are the ones that surprise you,
  and by then the rest works.

**Per feature**, a short block:

```
### F3 - Trip assignment
Covers: REQ-4, REQ-5 (AC-4.1 .. AC-5.3)
Prisma:     Trip.driverId, Trip.assignedAt, TripStatus enum + ASSIGNED
Shared:     schemas/trip.ts -> AssignTripInput
API:        POST /api/v1/trips/:id/assign   (role: operator, admin)
Service:    trips.service.ts -> assignDriver()
Rules:      only from status NEW; driver must be active; sets assignedAt
Web:        features/trips/TripBoardPage.tsx - assign button + driver picker
Test:       assign from NEW works; assign from ASSIGNED returns 409
Estimate:   2.5 days
```

**Data model decisions** that were not obvious, with the reason. Example: why
`price` is `Int` and not `Decimal`, why a status is an enum and not a string.

**Integration surface.** What this project touches that is not its own code:
external APIs, cron jobs, email, WhatsApp links, file storage. For each, what
happens when it is down.

**Risks.** The three things most likely to take longer than estimated, and what
you will do if they do.

**Rollback.** For each risky feature, how to turn it off without breaking the
rest. Usually a feature flag in config.

## Traceability check

Before finishing:

- Every `REQ-n` in the spec appears in exactly one feature block.
- Every feature block cites at least one `REQ-n`.
- The sum of feature estimates matches `estimate.md` within 15 percent. If not,
  one of the two documents is wrong - fix it now, not later.

## Do not

- Do not choose new libraries here. The stack is fixed in `rules/stack.md`. If
  something genuinely needs a new library, write it in a `Stack overrides`
  section with the reason, and add it to `stack_overrides` in `state.md`.
- Do not design abstractions for features that are not in the spec.

## Next step

Run `/stack:scaffold`.
