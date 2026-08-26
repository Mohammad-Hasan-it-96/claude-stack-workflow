# Lifecycle protocol (binding)

## Stages

```
intake -> spec -> estimate -> [GATE: client approval] -> plan -> scaffold
       -> feature (repeat) -> review -> ship
```

| Stage | Command | Writes code? | Produces |
|---|---|---|---|
| intake | `/stack:intake` | No | `_project/intake.md` |
| spec | `/stack:spec` | No | `_project/spec.md` |
| estimate | `/stack:estimate` | No | `_project/estimate.md` |
| plan | `/stack:plan` | No | `_project/plan.md` |
| scaffold | `/stack:scaffold` | Yes | the monorepo skeleton |
| feature | `/stack:feature <name>` | Yes | one vertical slice |
| review | `/stack:review` | No | `_project/review-<n>.md` |
| ship | `/stack:ship` | Yes | Docker, env, deploy notes |

## State

State lives in one file: `_project/state.md` front-matter. It is the only source
of truth for which stage the project is in. Never infer the stage from which
files exist on disk.

```yaml
---
project: taxi-office
stage: spec
approved_by_client: false
stack_overrides: []
features_done: [auth, drivers]
features_todo: [trips, dispatch, settlement]
updated: 2026-08-26
---
```

Only the command that owns a stage may change `stage`. A command must make one
edit to the front-matter and touch nothing else in the file.

## The one gate

`plan`, `scaffold`, and `feature` must refuse to run while
`approved_by_client: false`.

This is the whole point of the plugin. The most expensive mistake in freelance
work is writing code before the client agreed in writing on scope and price. If
the user asks to skip the gate, do it - but say once, in one sentence, what they
are giving up, then proceed without repeating it.

## Non-mutating stages

`intake`, `spec`, `estimate`, `plan`, and `review` must not create or edit any
file outside `_project/`. If one of them wants to change source code, it has
misunderstood its job.

## Feature loop

`/stack:feature` is called once per feature and runs the full vertical slice.
After each feature:

1. The app must run.
2. The feature must be usable in the browser.
3. `features_done` gains one entry and `features_todo` loses one.

If a feature cannot be finished, do not leave it half-built and move on. Say so,
and either finish it or revert it.

## Scope changes

When the client asks for something not in `spec.md`:

1. Do not just build it.
2. Add it to `_project/change-requests.md` with a day estimate and a price.
3. Tell the user what it costs before writing code.

Unpaid scope creep is the second most expensive mistake in freelance work.
