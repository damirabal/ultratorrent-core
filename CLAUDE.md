# Working in this repository

## Deploying — read [docs/OPERATIONS.md](docs/OPERATIONS.md) first

Never improvise a deploy. The procedure and the failures behind it are documented;
the rules below are the ones that have actually broken production, and a local
`PreToolUse` hook blocks each of them.

- **Build images through `ops/scripts/docker-build.sh`** (or the build host's deploy
  wrapper) — never a bare `docker compose build`. A bare build leaves `gitSha` null,
  so nothing can tell you what is running.
- **Never build on the constrained NAS host.** Build once on the build host and ship
  the image via its registry.
- **`npm run release:apply` requires `--no-git`.** Without it the script finalises
  with `git commit -a`, sweeping unrelated work-in-progress into the release commit
  and pushing it.
- **Verify with `build-info.json`**, not container uptime and not image IDs — a
  registry round-trip changes the image ID for identical content.
- **Prune on every deploy.** Build cache once filled the root disk and crash-looped
  Postgres mid-deploy.

Real host names, addresses and paths are **not** in this repository (it is public).
They live in `ops/hosts.local.md`, which is gitignored.

## The working tree is shared

Someone edits this tree live while you work, and the frontend serves from Vite dev
with HMR, so they verify UI changes without committing. A clean `git status` at the
start of a session does not mean it stays clean.

**Stage by explicit path — never `git add -A`, `git add .`, or `git commit -a`.**
Before committing, run `git status --short`; if unexpected modified files appear,
surface them rather than assuming they are yours.

## Before releasing

Four gates, each catching what the others miss — see
[docs/OPERATIONS.md](docs/OPERATIONS.md#pre-release-gates):

1. `npx tsc --noEmit -p apps/backend/tsconfig.json`
2. `cd apps/frontend && npx tsc` — the root `--noEmit` does **not** enforce
   `noUnusedLocals`, but the production build does
3. `npm test --workspace @ultratorrent/backend`
4. **A fresh build + boot** — NestJS DI and module-wiring errors throw only at
   bootstrap, and the dev box hides them behind a stale `dist/`

## Versioning

Changeset-driven, one canonical version. Author a changeset with every app change
(`npm run changeset:add -- --level <patch|minor|major> --summary "…"`). Cut a release
only on an explicit request. See [docs/VERSIONING.md](docs/VERSIONING.md).

## Architecture doc

Update [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and append a dated Change Log row
whenever you make an architectural change. A `Stop` hook checks this.
