# Model and cost policy (binding)

Thinking is cheap. Reading and writing files is expensive. This policy exists so
a full project does not burn the budget on boilerplate.

## The rule in one line

**Decide with the strong model in the main thread. Write files with a cheap
model in a subagent. Never let generated code enter the main context.**

## Model per stage

| Stage | Runs where | Model | Effort | Why |
|---|---|---|---|---|
| intake | main thread | inherit | medium | A conversation. Almost no file reading. Cheap already. |
| spec | main thread | inherit | high | Highest-value thinking in the whole project. Do not economise here. |
| estimate | main thread | inherit | medium | Text only. |
| plan | main thread | inherit | high | Every later cost is decided here. |
| scaffold | 4 parallel `codegen` agents | haiku | low | Pure boilerplate from a known template. No judgement needed. |
| feature | 1 `feature-builder` agent | sonnet | medium | Needs judgement, but the pattern is already fixed by the scaffold. |
| review | parallel reviewer agents | sonnet / inherit | high | Read-only, returns a short findings list. |
| ship | 1 `codegen` agent | haiku | low | Dockerfile, compose, nginx, env. Fully mechanical. |
| explain | main thread | inherit | low | Short answer, no file writing. |

## Context discipline - this saves more than model choice

1. **Never read a file you just wrote.** The write tool already confirmed it. Do
   not "verify" by reading it back.

2. **Never let a subagent return file contents.** A subagent that writes code
   returns a list of file paths and a one-line summary. Nothing else. If it
   returns code, the token saving is gone.

3. **Read sections, not files.** Once `spec.md` is long, pull the one feature
   block you need with Grep and a line range. Do not read the whole spec for
   every feature.

4. **Grep before Read.** Find the line first, then read 30 lines around it.
   Reading a 600-line file to change 3 lines is the most common waste.

5. **One feature per command run.** `/stack:feature trips` then
   `/stack:feature dispatch`. Never build three features in one run - the
   context grows and every later token costs more.

6. **Do not re-explain the stack.** It is in `rules/stack.md`. Subagents read it
   themselves. Do not paste it into a prompt.

## Speed - run independent work in parallel

Send multiple agents in **one message** so they run at the same time.

- **Scaffold**: the API core, the auth module, the web shell, and the shared
  package have no dependency on each other's contents once the plan fixes the
  interfaces. Four agents, one message.
- **Feature**: the Prisma model must exist before the service. But the API tests
  and the React page can be written in parallel once the endpoint contract is
  fixed. Two agents, one message.
- **Review**: every reviewer lens is independent. All of them in one message.

Do not parallelise work that shares a file. Two agents editing `app.ts` at the
same time will lose one of the edits.

## When to spend more, not less

Cheap models are wrong for these. Use the strong model:

- Anything touching money, permissions, or authentication.
- The state machine of the core object.
- A bug you have already failed to fix once.
- The spec and the plan, always.

Saving 20 cents on an authorization check is not a saving.
