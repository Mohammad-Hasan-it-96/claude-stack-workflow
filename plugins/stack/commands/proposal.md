---
description: Generate a client-facing proposal from PROJECT.md - phases, prices, timeline, payment terms, out of scope. Run only when you actually need something to send the client.
argument-hint: [language]
allowed-tools: Read, Write, Glob, Grep
---

# /stack:proposal

Write `proposal.md` for the client. This is one of only two files in this
workflow that a person outside the project reads, which is why it exists at all.

Write it in `$1` if given, otherwise the language the user has been speaking.

## Source

`PROJECT.md` only. If it does not exist, stop and say to run `/stack:scope`.

## Rules

**Include:** what they get, the phases, the price per phase, the timeline,
payment terms, and the out-of-scope list.

**Exclude:** your day rate, the learning tax, internal risk notes, and anything
in the Decisions table. Those are yours.

Write for a business owner, not a developer. No library names, no architecture,
no "REST API". Say what they will be able to do on which screen.

## Shape

```
# <Project> - Proposal

## What this system does
Three sentences in their words.

## What you get

### Phase 1 - <name>          <price>    <weeks>
- <feature in business language>
- <feature>

### Phase 2 - <name>          <price>    <weeks>
- ...

### Monthly maintenance       <price>/month
Hosting, bug fixes, small support.

## Not included
- <copied from Out of scope, in plain language>

## Payment
40% to start · 30% at the working demo · 30% at handover

## Terms
- Two rounds of revisions included
- One month of free support after handover
- <who pays hosting, domain, store accounts>
- Anything not listed above is a change request with its own price

## Timeline
Phase 1 starts <date>, ready in <n> weeks.
```

## Phases

Split the features from `PROJECT.md` into three tiers:

- **Phase 1** - the smallest set that replaces the current manual work
- **Phase 2** - what makes it comfortable to use daily
- **Phase 3** - priced, but sold later

Always price monthly maintenance separately. It is the income that continues
after the project ends.

## After writing

Tell the user in one line where it is and what to do when the client agrees:

```
proposal.md written. When the client agrees in writing, set approved: true in PROJECT.md.
```
