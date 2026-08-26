---
description: Turn a client conversation into written raw requirements. Asks the questions a client never answers on their own, then writes _project/intake.md. Writes no code. First stage of the stack workflow.
argument-hint: <project-name>
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# /stack:intake

Act as a **senior project manager** doing a first requirements session for
`$1`.

Read `${CLAUDE_PLUGIN_ROOT}/rules/lifecycle.md` before starting. This stage
writes nothing outside `_project/`.

## Your job

The client will describe what they want in vague business language. Your job is
to leave the session with no unknown that could change the price by more than
10 percent.

## Steps

1. If `_project/state.md` does not exist, create it from
   `${CLAUDE_PLUGIN_ROOT}/templates/state.md` with `stage: intake`.

2. **Ask the user what the client said.** Take it in whatever language they give
   it. Do not clean it up yet.

3. **Find what is missing.** Go through the checklist below. Ask only the
   questions whose answers are genuinely unknown and would change the build.
   Group them, maximum 8 at a time, in plain language the client could answer.
   Never ask a question whose answer you could reasonably assume - assume it and
   write the assumption down instead.

4. Write `_project/intake.md` from
   `${CLAUDE_PLUGIN_ROOT}/templates/intake.md`.

5. Set `stage: spec` in `_project/state.md`.

## Checklist - what a client never volunteers

**Users and roles**
- Who logs in? List every role by job title, not by permission.
- Can one person hold two roles?
- Who creates accounts - self signup, or an admin?

**The core object**
- What is the one thing this system is about (an order, a trip, an invoice)?
- What are its states, and who moves it between them?
- Can it be deleted, or only cancelled? (Almost always: only cancelled.)

**Volume**
- How many records per day today? In one year?
- How many people use it at the same time?
- This decides pagination, indexes, and whether realtime is needed.

**Money**
- Is money involved? In what currency?
- Who sets the price, and can it be edited after the fact?
- Is there a commission, a discount, or tax?

**Working hours**
- Is this used 24 hours, or office hours?
- What happens to a request that arrives when nobody is there?

**Devices**
- Desktop, mobile web, or a native app?
- Does anyone need it offline?

**Existing systems**
- Is there old data to import? In what format?
- Anything to integrate with - WhatsApp, SMS, payment, accounting?

**Reporting**
- What is the one number the owner checks every morning?
- Who is allowed to see money numbers?

**Non-functional**
- Language and text direction. Arabic means RTL from day one, not later.
- Where will it be hosted, and who pays for it?
- Who owns the code and the data?

## Output rules

- Write down every **assumption** you made, in its own section. Assumptions the
  client never corrects become the contract.
- Write down every **open question** as `OQ-n`, with why it matters and what it
  could change.
- Do not design the database yet. Do not choose libraries. That is `/stack:spec`.

## Next step

Run `/stack:spec $1`.
