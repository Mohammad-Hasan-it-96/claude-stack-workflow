---
description: Turn a client conversation into a complete PROJECT.md - requirements, roles, data model, states, build order - and give days and price in the chat. Replaces separate intake, spec, estimate, and plan steps. One command, one file.
argument-hint: <project-name>
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# /stack:scope

Act as a **senior project manager and architect at the same time**. Produce
everything needed to start building, in one pass.

Read first: `${CLAUDE_PLUGIN_ROOT}/rules/flow.md`

Output: **one file**, `PROJECT.md`. Days and price go in the chat, not in a
file. If the user wants something to send the client, they run
`/stack:proposal`.

## Step 1 - read what you already have

If the user has already described the project in this conversation, or there are
notes in the folder, use them. Do not re-ask what you were already told.

## Step 2 - judge the size, then ask ONE round of questions

| Size | Signal | Questions to ask |
|---|---|---|
| small | under 5 features, one role, no money | 3 |
| normal | 5-15 features, 2-4 roles, money | up to 8 |
| large | 15+ features or an external integration | up to 8, plus the integration |

Put them all in one `AskUserQuestion` call. Ask only what changes the build or
moves the price by more than 10 percent. Everything else becomes an assumption
you write down.

The questions that actually matter, in order of how often they change the price:

1. Who logs in, and what are the roles?
2. What is the core object, what are its states, and who moves it between them?
3. Is money involved - who sets the price, is there a commission?
4. Records per day now, and in one year?
5. Working hours - and what happens to a request when nobody is there?
6. Language and direction (Arabic means RTL from day one, not later)?
7. Anything to integrate with - WhatsApp, SMS, payment, an old system?
8. Who pays for hosting, and who owns the code?

Do not ask about libraries, architecture, or hosting provider. Those are
decided in `rules/stack.md`.

## Step 3 - write PROJECT.md

From `${CLAUDE_PLUGIN_ROOT}/templates/PROJECT.md`. Fill only the sections that
have content. Delete empty ones.

Keep it tight. A normal project is 80 to 150 lines. If it passes 200, you are
writing a document instead of a plan.

**Requirements** - `R1`, `R2`, one sentence each, with the acceptance criterion
on the same line. Not a separate numbered list of ACs. If a requirement needs
"and" twice, split it.

**Data model** - a compact code block, not a table per model. Field names and
types on one line each.

**State transitions** - a table. Every move not in it is refused with 409.

**Features** - in the front-matter, with `days` and `req` on each. This list is
the build order and the estimate at the same time. Order it:

1. auth and roles first
2. the core object, simplest flow, end to end
3. states and transitions
4. money and settlement
5. reports
6. realtime, uploads, integrations last - they are the ones that surprise you

**Out of scope** - name things explicitly. This section protects the price.

## Step 4 - estimate, in the chat

Sum the feature days. Then apply, in order:

| Multiplier | Rule |
|---|---|
| Learning tax | +30% on a feature using a pattern the user has not built before |
| Integration tax | double anything calling another company's API |
| Testing and fixing | +20% of subtotal |
| Communication | +10% of subtotal |

Give the total as a **range**, and a price at their day rate. Then sanity-check
out loud: price per developer per day under 10 dollars is a loss, not a project.
Say so plainly if it happens.

Show it as a short table in the chat. Do not write an estimate file.

## Step 5 - say what is next, in one line

```
PROJECT.md written - 9 features, 24-31 days.
Next: /stack:proposal  (client document)   or   /stack:scaffold  (start building)
```

## Do not

- Do not create `intake.md`, `spec.md`, `plan.md`, or a research file.
- Do not ask a second round of questions. Assume, write it down, move on.
- Do not design the API endpoint list here. `/stack:feature` derives it from the
  data model when it needs it.
- Do not choose libraries. The stack is fixed.
