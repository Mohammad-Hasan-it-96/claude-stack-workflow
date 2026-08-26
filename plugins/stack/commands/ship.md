---
description: Prepare the project for a real VPS - production Docker build, nginx, HTTPS, backups, env checklist, and a handover document for the client. Delegates the mechanical files to a cheap codegen agent.
argument-hint: [domain]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task
---

# /stack:ship

Get the project onto a server and into the client's hands.

Read first: `${CLAUDE_PLUGIN_ROOT}/rules/model-policy.md`

## Step 1 - refuse to ship broken work

Run these first. If any fails, stop and say what failed. Do not ship around it.

```
npx tsc --noEmit
npm run test -w apps/api
npm run build
```

Also check, by reading and not by guessing:
- `.env` is in `.gitignore` and is not committed
- `.env.example` lists every variable the app reads
- the seeded default admin password is not still the default
- `NODE_ENV=production` disables stack traces in the error middleware
- CORS has an explicit origin, not `*`

## Step 2 - spawn one codegen agent for the mechanical files

```
Dockerfile                      apps/api - multi-stage, node:20-alpine
Dockerfile                      apps/web - build then serve with nginx
docker-compose.prod.yml         db + api + web + nginx, with volumes
nginx/default.conf              /api to api:3000, everything else to web
.env.production.example
scripts/backup-db.sh            pg_dump to a dated file, keep 14 days
scripts/deploy.sh               pull, build, migrate, restart
```

Tell it the domain, the ports, and the database name. It returns a file list.

## Step 3 - write the deploy runbook yourself

Append a `## Deploy` section to the project's own `README.md` - the file the
user will actually open in six months. Short and copy-pasteable:

- VPS size that fits the expected volume from `PROJECT.md`
- the exact commands, in order, from a fresh Ubuntu box
- how to get HTTPS (certbot), and how it renews
- how to restore a backup - untested backups are not backups
- how to read logs, and how to restart one service
- what to do when the disk fills up

## Step 4 - write the client handover document

`handover.md`, in the client's language, no technical words. This is one of the
only two files this workflow writes for a human outside the project:

- how to log in, and the admin account
- how to add a user and change a role
- the daily tasks, with the screen name for each
- what to do when something looks wrong, and how to reach you
- what is covered by the free support period and what is not
- what is out of scope, copied from the Out of scope section of `PROJECT.md`

This document ends most support calls before they happen.

## Step 5 - the delivery checklist

Report it filled in, not as a template:

```
[ ] Client approved the spec in writing
[ ] Final payment terms agreed
[ ] Domain and hosting paid, and in whose name
[ ] Backups running and one restore tested
[ ] Admin password changed from the seed
[ ] Client trained, handover.md delivered
[ ] Free support period start date recorded
[ ] Source code delivered or access granted, per the contract
```

## Never do without asking

- Never run `deploy.sh` against a live server on your own.
- Never run a migration on production data without a fresh backup.
- Never push secrets to a git remote.

Ask first, every time, even if you asked yesterday.
