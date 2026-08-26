---
description: Show where the project stands - stage, approval, features done and remaining, open change requests, and the next command to run. Reads only the state file. Cheap.
argument-hint:
allowed-tools: Read, Glob, Grep
---

# /stack:status

Show where the project is. Read `_project/state.md` and nothing else unless a
number is missing.

This command must stay cheap. Do not scan the repository. Do not read `spec.md`
or `plan.md` to count things - the counts live in `state.md`, and if they are
wrong the fix is to correct `state.md`, not to recount every time.

## Output

```
PROJECT   taxi-office
STAGE     feature
APPROVED  yes  (2026-08-20)

DONE      auth, drivers, customers          (3)
TODO      trips, dispatch, settlement       (3)

CHANGE REQUESTS   1 open, 2 days, not yet priced to client
STACK OVERRIDES   none

NEXT      /stack:feature trips
```

## Rules

- If `_project/state.md` does not exist, say so and point to `/stack:intake`.
- If `approved_by_client` is `false` and the stage is past `estimate`, say it in
  one line. Once. Do not lecture.
- If `change-requests.md` has an open item that was never priced to the client,
  flag it - that is unpaid work in progress.
- Always end with the exact next command to run.
