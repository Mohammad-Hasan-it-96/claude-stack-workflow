---
description: Run parallel read-only review agents (security, senior engineering) over the current diff or a named feature, verify their findings, and report a short ranked list. Writes no source code.
argument-hint: [feature-name | --diff]
allowed-tools: Read, Grep, Glob, Bash, Write, Task
---

# /stack:review

Review `$1`. With no argument or with `--diff`, review the uncommitted changes.

Read first: `${CLAUDE_PLUGIN_ROOT}/rules/model-policy.md`

This command writes nothing outside `_project/`. It does not fix anything - the
user decides what to fix.

## Step 1 - define the target, cheaply

```
git diff --stat
git diff --name-only
```

Pass the **file list** to the agents, not the diff contents. They read what they
need themselves. Pasting a large diff into two agent prompts pays for it twice.

If the target is a feature name, list its files with Glob instead.

## Step 2 - spawn both reviewers in ONE message

- `security-reviewer` - authorization, validation, secrets, injection
- `senior-reviewer` - over-engineering, pattern fit, layer leaks, broken flows

They are independent and read-only, so they run at the same time.

Give each the same short prompt: the file list, the feature name, and the role
matrix rows from `spec.md` that apply. Nothing else.

## Step 3 - verify before reporting

An agent finding is a claim, not a fact. For each one, check the actual file
yourself with a small targeted read. Drop any finding where:

- the code does not say what the agent said it says
- a middleware already handles it, and the agent did not read that middleware
- it is a style opinion
- you cannot write the concrete failure in one sentence

Do not report a finding you did not verify. A false finding costs the user more
time than a missed one.

## Step 4 - report

Ranked, most severe first, with `file:line`:

```
REVIEW - trips feature

1. [CRITICAL] apps/api/src/modules/trips/trips.service.ts:42
   Operator of office A can read trips of office B.
   findUnique filters by id only. Add officeId from the token.

2. [MAJOR] apps/api/src/modules/trips/trips.controller.ts:18
   Status transition check sits in the controller, so the assign path skips it.

3. [MINOR] ...

Verified 3 of 6 agent findings. 3 dropped as not real.
```

Write the same list to `_project/review-<n>.md`.

If nothing survived verification, say exactly: `No real findings.` Do not pad
the report to look thorough.

## Step 5 - offer, do not act

Ask which findings to fix. Fix only those. Fixing everything an agent reports,
without being asked, is how a review turns into an unplanned refactor.

## When to run this

- After every feature that touches money, permissions, or the state machine.
- Once before handing the project to the client.
- Not after every small change - it costs real tokens and finds nothing new.
