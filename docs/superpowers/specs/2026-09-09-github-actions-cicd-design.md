# Design: GitHub Actions CI + placeholder CD (docs first)

Date: 2026-09-09  
Status: approved for documentation delivery

## Goal

Define how Draw The World uses GitHub Actions for continuous integration and a placeholder continuous deploy on `main`, documented before any workflow YAML exists.

## Decisions (from brainstorming)

| Topic | Choice |
|-------|--------|
| Scope | CI + CD (not CI-only) |
| Delivery order | Docs first; workflow implementation later |
| Deploy target | Placeholder / reserved secrets; provider TBD |
| Triggers | `push` to `main` only (no PR checks) |
| Doc shape | ADR for *why* + `docs/ci/mvp.md` for *conventions* |

## Artifacts

1. [`docs/adr/0003-github-actions-main-cicd.md`](../../adr/0003-github-actions-main-cicd.md) — decision record  
2. [`docs/ci/mvp.md`](../../ci/mvp.md) — pipeline shape, jobs, secrets, scaffold no-op, monorepo mapping  
3. Cross-link from `.scratch/mvp/issues/11-runbook-seed.md` Comments → `docs/ci/mvp.md` (runbook later points at CI)

## Out of scope this delivery

- Creating `.github/workflows/ci-cd.yml`
- Choosing Fly/Railway/VPS/etc.
- PR status checks, EAS/store publish, auto migrate on production

## Implementation follow-up

After this spec is accepted, write an implementation plan (writing-plans) only when ready to add the workflow file that realizes `docs/ci/mvp.md`.
