---
project: <name>
stage: scope
approved: false
day_rate:
features:
  - { id: F1, name: auth, days: 5, status: todo, req: [R1] }
  - { id: F2, name: trips, days: 3, status: todo, req: [R2, R3] }
updated: <YYYY-MM-DD>
---

# <project>

<Three sentences: what it does, for whom, what manual work it replaces.>

## Requirements

- **R1** <one sentence> — AC: <what you click to prove it works>
- **R2** <one sentence> — AC: <...>

## Roles

| Action | Admin | Operator | Customer |
|---|---|---|---|
| | | | |

## Data model

```
Trip    id, customerId, driverId?, status:TripStatus, priceMinor:Int?,
        pickupText, destText, createdAt, updatedAt, deletedAt?
Driver  id, name, phone, carNumber, active:Bool
TripStatus  NEW ASSIGNED IN_PROGRESS COMPLETED PAID CANCELLED
```

Money is an integer in the smallest unit. Times are UTC.

## State transitions

| From | To | Who | Rule |
|---|---|---|---|
| NEW | ASSIGNED | operator | driver active |

Anything not listed here is refused with 409.

## Decisions

| Decision | Why |
|---|---|

## Out of scope

- <named explicitly — this is what protects the price>

## Changes after approval

| Date | What | Days | Priced? |
|---|---|---|---|

---

*This is the only project file. Requirements, plan, and state live here
together. If a section is empty, delete it.*
