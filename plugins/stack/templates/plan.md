# Build plan - <project-name>

Date: <YYYY-MM-DD>
Based on: spec.md v1.0, estimate.md

## Build order

| # | Feature | Why here |
|---|---|---|
| F1 | Scaffold + auth + roles | everything depends on knowing who is asking |
| F2 | Core object, simplest flow | proves the stack end to end |
| F3 | States and transitions | |
| F4 | Money and settlement | |
| F5 | Reports | |
| F6 | Realtime / uploads / integrations | last - these are the ones that surprise you |

## Features

### F3 - <name>

```
Covers:     REQ-4, REQ-5 (AC-4.1 .. AC-5.3)
Prisma:     Trip.driverId, Trip.assignedAt, TripStatus + ASSIGNED
Shared:     schemas/trip.ts -> AssignTripInput
API:        POST /api/v1/trips/:id/assign   roles: operator, admin
Service:    trips.service.ts -> assignDriver()
Rules:      only from NEW; driver must be active; sets assignedAt
Errors:     TRIP_ALREADY_ASSIGNED 409, DRIVER_INACTIVE 422
Web:        features/trips/TripBoardPage.tsx - assign button + driver picker
Empty:      "No trips yet today"
Test:       assign from NEW works; assign from ASSIGNED returns 409
Estimate:   2.5 days
```

## Data model decisions worth recording

| Decision | Reason |
|---|---|
| priceMinor is Int, not Decimal | JavaScript has one float number type; money must be an integer |
| status is an enum, not a string | invalid states become impossible, not just unlikely |

## Integration surface

What this project touches that is not its own code.

| Thing | Used for | What happens when it is down |
|---|---|---|

## Stack overrides

Anything not in `rules/stack.md`, with the reason. Empty is the normal case.

| Library | Instead of | Reason |
|---|---|---|

## Risks

| # | Risk | If it happens |
|---|---|---|
| R-1 | | |

## Rollback

| Feature | How to turn it off without breaking the rest |
|---|---|

## Traceability check

- [ ] Every REQ appears in exactly one feature block
- [ ] Every feature block cites at least one REQ
- [ ] Sum of feature estimates matches estimate.md within 15%
