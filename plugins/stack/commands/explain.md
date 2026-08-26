---
description: Explain one JavaScript or Node concept to a Laravel developer, anchored to its Laravel equivalent. Short answer, real example from this codebase when one exists. Writes nothing.
argument-hint: <topic>
allowed-tools: Read, Grep, Glob
---

# /stack:explain

Explain `$1` to an experienced Laravel developer who is new to this ecosystem.

Read `${CLAUDE_PLUGIN_ROOT}/rules/laravel-map.md` first.

## Who you are talking to

A senior PHP developer. They understand MVC, dependency injection, migrations,
middleware, queues, and ORMs. They do **not** need programming explained. They
need the *translation* and the *differences that bite*.

Never write "a function is a block of code that...". Never define what an API
is. If you find yourself explaining a concept they clearly already have in
Laravel, you have misread the question - answer the ecosystem difference
instead.

## Shape of the answer

Keep it under 40 lines. Four parts, in this order:

1. **The Laravel equivalent**, in one sentence.
   > `asyncHandler` is what Laravel's exception handler does for you automatically.

2. **The difference that matters**, in two or three sentences. This is the part
   worth reading. What breaks, when, and why it is not obvious.

3. **A real example from this codebase** if one exists. Grep for it and cite
   `file:line`. If the project has not been scaffolded yet, write a six-line
   example instead. Never more than fifteen lines of code.

4. **The trap.** The mistake a Laravel developer specifically makes here. One
   sentence. If there is no real trap, leave this part out rather than inventing
   one.

## Rules

- One concept per answer. If the question contains three, answer the most useful
  one and list the other two as follow-up commands.
- No links to documentation unless the user asks. They can search.
- Simple English, short sentences, common words.
- If you are not sure the concept works the way you remember, check the code in
  the repo or say you are not sure. Do not guess about behaviour the user will
  rely on.

## Good topics

`async/await`, `promises`, `asyncHandler`, `prisma include`, `prisma vs eloquent`,
`zod`, `jwt vs sanctum`, `npm workspaces`, `esm vs commonjs`, `vite proxy`,
`tanstack query`, `react state`, `useEffect`, `axios interceptor`, `typescript
generics`, `null vs undefined`, `event loop`.
