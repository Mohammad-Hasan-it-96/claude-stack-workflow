---
description: Produce an honest day estimate, a phase breakdown, and a freelance price range from the spec. Writes _project/estimate.md plus a client-facing proposal. Writes no code.
argument-hint: <project-name>
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /stack:estimate

Act as a **senior engineer who has been burned by underquoting**. Turn
`_project/spec.md` into days and money.

This stage writes nothing outside `_project/`.

## Steps

1. Confirm `_project/spec.md` exists.
2. Estimate per feature, not per layer.
3. Write `_project/estimate.md` (internal, has the real numbers).
4. Write `_project/proposal.md` (client-facing, no internal day rates).
5. Set `stage: plan` in `_project/state.md` and leave
   `approved_by_client: false`.

## How to estimate

Estimate each feature as a **vertical slice**: Prisma model, migration, Zod
schema, service, controller, route, API hook, page, and a manual test.

Base rates for this stack, for a developer at intermediate level:

| Work item | Days |
|---|---|
| Project scaffold, auth, roles, layout, deploy skeleton | 4 - 6 |
| Simple CRUD feature (one table, list + form) | 1 - 1.5 |
| CRUD with relations, filters, and a status field | 2 - 3 |
| A screen with real business logic (assignment, settlement, pricing) | 3 - 5 |
| Realtime screen (Socket.IO, live list) | 2 - 3 |
| Report or dashboard with charts | 2 - 3 |
| File upload plus storage | 1 - 2 |
| Integration with an external service | 2 - 4, and never trust the low end |
| Docker, VPS deploy, domain, HTTPS, backups | 2 - 3 |
| Client training and handover docs | 1 - 2 |

Then apply, in this order:

1. **Learning tax.** If the user has not built this pattern in this stack
   before, add 30 percent to that feature. Say which features got the tax and
   why. This is honesty, not padding.
2. **Integration tax.** Anything involving another company's API: double it.
3. **Testing and bug fixing.** Add 20 percent of the subtotal.
4. **Communication.** Add 10 percent. Meetings and messages are real hours.

State the total as a **range**, low to high. Never a single number.

## How to price

```
price = total_days x day_rate
```

Then sanity-check against these, and say so if they disagree:

- Compare to what the same system costs as a ready product for one year.
- Compare to the client's own cost of the manual process it replaces.
- A price under 10 dollars per day per developer is not a project, it is a loss.
  Say this plainly if it happens.

Present **three tiers**:

- **Phase 1 (MVP)** - the smallest thing that replaces the current manual work.
- **Phase 2** - the features that make it comfortable.
- **Phase 3** - the nice-to-haves. Priced, but sold later.

Always add **monthly maintenance** as a separate line: hosting, bug fixes, small
support. This is the income that lasts after the project ends.

## Payment terms to include in the proposal

- 40 percent up front, 30 percent at the working demo, 30 percent at handover.
- Number of free revision rounds (suggest two).
- Free support period (suggest one month).
- Who pays for hosting, domain, and store accounts.
- Anything outside `spec.md` is a change request with its own price.

## What must be in the client-facing proposal

Include: what they get, the phases, the price per phase, the timeline, payment
terms, and the out-of-scope list.

Exclude: your day rate, the learning tax, and any internal risk note. Those live
in `estimate.md` only.

## Next step

Send `proposal.md` to the client. When they agree in writing, set
`approved_by_client: true` in `_project/state.md`, then run `/stack:plan $1`.
