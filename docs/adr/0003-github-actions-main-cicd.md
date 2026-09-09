# GitHub Actions on `main` for CI + placeholder CD

Delivery uses GitHub Actions. For MVP we run continuous integration and a placeholder continuous-deploy job only on pushes to `main`. Pull-request checks are deferred. The deploy target (PaaS, VPS, etc.) is intentionally unset; secrets and Environment names are reserved so a later ADR can bind a real provider without renaming the pipeline shape.

## Considered Options

- **Actions on `main` only, CD placeholder (chosen)**: matches current empty/scaffold repo; avoids PR gate noise before monorepo exists; keeps one workflow story for agents and humans via `docs/ci/mvp.md`.
- **PR CI + `main` CD**: stronger merge hygiene, but no app code yet and no branch protection requirement for MVP docs stage.
- **Manual `workflow_dispatch` CD only**: safest deploys, but weaker “merge to main ships” habit and easy to forget once a provider exists.
