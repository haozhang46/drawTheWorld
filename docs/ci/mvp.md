# CI / CD — MVP conventions

Source of truth for pipeline *shape*. Decision rationale: [`docs/adr/0003-github-actions-main-cicd.md`](../adr/0003-github-actions-main-cicd.md).  
Monorepo expectations: [`docs/superpowers/plans/2026-09-09-mvp-draw-the-world.md`](../superpowers/plans/2026-09-09-mvp-draw-the-world.md) (pnpm 9+, Node 22+, Turborepo, `apps/api`, `apps/mobile`, `packages/domain`).

This document does **not** require `.github/workflows` to exist yet. Implementers must follow this file when adding workflows.

## Trigger

- Event: `push` to `main` only.
- No pull-request workflows in MVP.
- Optional later: `workflow_dispatch` for dry-run deploy — not required for v1 of the workflow.

## Workflow file (when implemented)

- Path: `.github/workflows/ci-cd.yml`
- Jobs: `ci` → `deploy` (`deploy` has `needs: [ci]`)

## Job: `ci`

| Step | Convention |
|------|------------|
| Runtime | Node 22; enable pnpm 9+ (Corepack or `pnpm/action-setup`) |
| Install | `pnpm install --frozen-lockfile` when `pnpm-lock.yaml` exists |
| Checks | lint, typecheck, unit tests for packages that exist (`@dtw/domain`, `@dtw/api` via turbo/filter scripts once defined in root `package.json`) |
| Mobile | Scriptable typecheck/lint only when `apps/mobile` exists; **no** Maestro/Detox, **no** store/EAS build in MVP CI |
| Services | Postgres 16 / MinIO as GitHub Actions service containers **only** if API tests need them; pure unit tests may skip |
| Scaffold / empty tree | If root `package.json` is missing, `ci` exits successfully after a documented no-op (log: “monorepo not scaffolded”) so docs-only `main` stays green |

Failure of any real check fails the workflow; `deploy` must not run.

## Job: `deploy`

| Rule | Convention |
|------|------------|
| Gate | `needs: ci` |
| Environment | GitHub Environment name: `production` |
| Behavior (MVP) | Placeholder only: echo intended target, assert required secret *names* are present (or explicitly `unset`), do **not** mutate infrastructure |
| Provider | TBD — bind real commands in a follow-up ADR; keep job id `deploy` |
| Rollback | None while placeholder |
| Migrate | Production DB migrate is **not** auto-run in MVP CD; decide in a later ADR / runbook |
| Mobile release | Out of scope (no App Store / Play / EAS submit) |

## Secrets (names reserved)

Set on the repo or on Environment `production`. Until a provider is chosen, values may be placeholders.

| Name | Purpose |
|------|---------|
| `DEPLOY_PROVIDER` | String identifying provider; use `unset` until chosen |
| `DEPLOY_TOKEN` | Generic deploy credential; map to the real provider token later |
| `DEPLOY_API_URL` | Optional health-check base URL after a real deploy exists |

Do not invent alternate secret names in the workflow without updating this table.

## Mapping to monorepo packages

| Path | CI expectation |
|------|----------------|
| `packages/domain` | Unit tests + typecheck when present |
| `apps/api` | Unit/integration tests + typecheck/lint when present |
| `apps/mobile` | Typecheck/lint when present; native AR module not built in CI |
| `docker-compose.yml` | Local/dev; CI uses service containers only if tests require them |

Root scripts (to be added with Task 2+) should expose stable names the workflow can call, e.g. `pnpm lint`, `pnpm typecheck`, `pnpm test` (or turbo pipeline equivalents). Prefer documenting those names here when they land.

## Non-goals (MVP)

- Branch protection / required PR status checks
- Preview environments per PR
- Automatic production migrations
- Mobile binary distribution
- Non-GitHub CI vendors
