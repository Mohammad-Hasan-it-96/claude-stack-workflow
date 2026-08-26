---
description: Build one feature as a complete vertical slice - Prisma model, migration, shared Zod schema, service, controller, route, API hook, React page, test. Delegates the writing to a mid-cost agent. Run once per feature.
argument-hint: <feature-name>
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task, AskUserQuestion
---

# /stack:feature

Build the feature `$1` end to end.

Read first: `${CLAUDE_PLUGIN_ROOT}/rules/model-policy.md`

## One feature per run

Do not build two features in one run, even if the user asks. Each one grows the
context and makes every later token cost more. Finish `$1`, report, and let the
user run the command again.

## Step 1 - get only what you need

Grep `_project/plan.md` for the feature block for `$1` and read **only** that
block, plus the lines around it. Do not read the whole plan. Do not read
`spec.md` unless the plan block references an `AC` whose text you actually need.

If there is no block for `$1`, stop and say so. Building a feature that is not
in the plan is unbilled work.

## Step 2 - decide the contract in this thread

Before spawning anything, fix these so the agent never has to guess:

- Prisma model name, fields with types, relations, enum values
- Which fields are nullable, and which are indexed
- The endpoint list: method, path, roles allowed
- The Zod schema names in `packages/shared`
- The error codes and when each fires
- The state transitions this feature allows, and from which states
- The pages, and what each shows when the list is empty

If any of these is genuinely unclear from the plan, ask the user **now**, in one
short group of questions. Asking after the code is written costs ten times more.

## Step 3 - spawn one feature-builder agent

Give it the contract from step 2 and the plan block. Tell it to copy the shape
of the existing `users` module.

Use two agents in one message **only** when the feature is large and the API
contract is fully fixed: one for the API side, one for the web side. They must
not both edit `app.ts` or `routes.tsx` - assign those files to one agent only.

For a feature that touches money, permissions, or the core state machine, do
**not** delegate the service file. Write that one yourself in this thread and
let the agent do the rest. Saving tokens on an authorization check is not a
saving.

## Step 4 - verify in this thread

```
npx tsc --noEmit
npm run test -w apps/api
npm run dev
```

Then open the page and do the main action once. A feature that compiles but
cannot be used is not done.

Fix failures with targeted edits: Grep for the symbol, read a small range, edit.
Never read a whole generated file. Never re-run the agent for a small fix.

## Step 5 - close out

Update `_project/state.md`: move `$1` from `features_todo` to `features_done`.
One edit to the front-matter, nothing else in the file.

## Report

Keep it short:

```
FEATURE $1 - done
Covers: REQ-4, REQ-5
Files: 11 written, 2 edited
Checks: tsc pass, 6 tests pass, page works
Next: /stack:feature <next from features_todo>
```

If the feature revealed something the spec got wrong, say it in one line and add
it to `_project/change-requests.md` with a day estimate. Do not silently absorb
extra scope.

## Teaching note

If this feature used a pattern the user has not met before, add two or three
sentences at the end explaining it against its Laravel equivalent. See
`${CLAUDE_PLUGIN_ROOT}/rules/laravel-map.md`. One concept per feature, maximum.
