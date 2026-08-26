# Specification - <project-name>

Version: 1.0
Date: <YYYY-MM-DD>
Based on: intake.md

## 1. Summary

Three sentences. What the system does, for whom, and what it replaces.

## 2. Requirements

### REQ-1 - <title>
<one sentence, business language>

- **AC-1.1** - <checkable by clicking>
- **AC-1.2** -

### REQ-2 - <title>

## 3. Roles and permissions

| Action | Admin | Operator | Accountant | Customer |
|---|---|---|---|---|
| Create trip | yes | yes | no | yes |
| Assign driver | yes | yes | no | no |
| See money totals | yes | no | yes | no |

This table becomes authorization code. Be exact.

## 4. Data model

### Model: <Name>

| Field | Type | Null | Notes |
|---|---|---|---|
| id | uuid | no | |
| status | TripStatus | no | enum below |
| priceMinor | Int | yes | integer, smallest currency unit |
| createdAt | DateTime | no | |
| updatedAt | DateTime | no | |
| deletedAt | DateTime | yes | soft delete |

Relations:
- `Trip.driver` -> `Driver` (many trips to one driver, nullable until assigned)

Enums:
- `TripStatus`: NEW, ASSIGNED, IN_PROGRESS, COMPLETED, PAID, CANCELLED

## 5. State machine

| From | To | Who | Rule |
|---|---|---|---|
| NEW | ASSIGNED | operator, admin | driver must be active |
| ASSIGNED | IN_PROGRESS | operator, admin | |
| IN_PROGRESS | COMPLETED | operator, admin | final price required |
| COMPLETED | PAID | accountant, admin | |
| NEW, ASSIGNED | CANCELLED | operator, admin, customer | reason required |

Any transition not in this table must be refused with 409.

## 6. API surface

| Method | Path | Purpose | Roles |
|---|---|---|---|
| POST | /api/v1/auth/login | | public |
| GET | /api/v1/trips | list, paginated | operator, admin |
| POST | /api/v1/trips/:id/assign | | operator, admin |

## 7. Error codes

| Code | Status | When |
|---|---|---|

## 8. Screens

### Operator
| Screen | Shows | Actions | Empty state |
|---|---|---|---|

## 9. Out of scope

Discussed but **not** being built. Named explicitly. This section protects the
price.

- <item>
- <item>

## 10. Non-functional

- Language and direction:
- Timezone:
- Browsers:
- Expected volume:
- Hosting:

## 11. Assumptions carried from intake

| # | Assumption |
|---|---|

## Checklist before this spec is finished

- [ ] Every AC is checkable by a person clicking, not by reading code
- [ ] Every screen maps to at least one REQ
- [ ] Every REQ maps to at least one screen or endpoint
- [ ] Every status in the state machine exists in the data model enum
- [ ] No requirement contains "etc.", "and so on", or "similar"
- [ ] No open questions remain
