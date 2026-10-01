# ci-templates

Reusable GitHub Actions workflows (`workflow_call`) for RIPRODUCTIONS repos.

| Template | Steps |
|---|---|
| `node-web.yml` | setup-node -> `npm ci` -> lint -> typecheck -> test -> build (each script optional, `--if-present`) |
| `pnpm.yml` | same as node-web with pnpm (`--frozen-lockfile`, optional `recursive`) |
| `expo.yml` | `npm ci` -> `tsc --noEmit` -> `expo-doctor` -> `expo export` for ios + android (Linux) |
| `python.yml` | uv venv -> install requirements/pyproject -> ruff -> pytest (`allow-no-tests`) -> `uv pip check` |
| `security.yml` | non-blocking `npm audit` / `pnpm audit` / `pip-audit`, results in the job summary |

Every template: `runs-on` input (JSON, default `"ubuntu-latest"`), `timeout-minutes` 15,
dependency caching, a random `PORT`, `concurrency` per ref (cancel-in-progress on PRs),
`permissions: contents: read`, and a `dummy-env` input (newline-separated `KEY=VALUE`
placeholders; never put real secrets there). No secrets are required.

## Caller example

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  ci:
    uses: RIPRODUCTIONS/ci-templates/.github/workflows/node-web.yml@main
    with:
      build-script: ""            # skip a step
      # runs-on: '["self-hosted","linux","x64"]'
      # dummy-env: |
      #   NEXTAUTH_SECRET=ci-dummy
  security:
    uses: RIPRODUCTIONS/ci-templates/.github/workflows/security.yml@main
```

The required status check name is `<caller job id> / <template job>`, e.g. `ci / node-web`.

## Access

This repo is private; Actions access is set to `user` so other RIPRODUCTIONS repos can call it:
`gh api -X PUT repos/RIPRODUCTIONS/ci-templates/actions/permissions/access -f access_level=user`.

Validate changes with `actionlint` before pushing.
