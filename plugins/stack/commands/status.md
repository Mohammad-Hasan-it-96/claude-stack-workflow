---
description: Show where the project stands - phase, approval, features done and left, unpriced changes, and the next command. Reads only PROJECT.md front-matter. Cheap by design.
argument-hint:
allowed-tools: Read, Glob, Grep
---

# /stack:status

Read the front-matter of `PROJECT.md`. Nothing else.

Do not scan the repository. Do not count files. Do not read the body of
`PROJECT.md` unless the `Changes after approval` table is needed. If a number in
the front-matter is wrong, the fix is to correct it once - not to recount every
time this command runs.

## Output

```
taxi-office          build          approved ✓

DONE  auth · drivers · customers                    13 days
TODO  trips · dispatch · settlement                 11 days

CHANGES  1 row not yet priced to client

NEXT  /stack:feature trips
```

Keep it to these lines. No headings, no explanation, no summary paragraph.

## Rules

- No `PROJECT.md` in the folder: say so, and point to `/stack:scope`.
- `approved: false` and `stage: build`: add one line saying the client has not
  approved in writing. Once. No lecture.
- A row in `Changes after approval` with an empty `Priced?` column is unpaid
  work in progress. Flag it - that is the one thing worth interrupting for.
- Always end with the exact next command, ready to copy.
