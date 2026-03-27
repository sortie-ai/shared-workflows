# shared-workflows

Reusable GitHub Actions workflows and composite actions for [sortie-ai](https://github.com/sortie-ai) repositories.

## Contents

```
.github/
├── actions/
│   └── dependabot-rebase/           # Composite action: check PR merge state, request rebase
└── workflows/
    ├── dependabot-merge.yml         # Auto-approve and merge Dependabot PRs after CI
    └── dependabot-rebase-sweep.yml  # Periodic scan: rebase stale Dependabot PRs
```

## Quick start

### Dependabot auto-merge

In your repository, create `.github/workflows/dependabot-merge.yml`:

```yaml
name: Dependabot auto-merge

on:
  workflow_run:
    workflows: [CI]
    types: [completed]

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    uses: sortie-ai/shared-workflows/.github/workflows/dependabot-merge.yml@v1
    secrets:
      bot-token: ${{ secrets.CICDBOT_TOKEN }}
```

### Dependabot rebase sweep

```yaml
name: Dependabot rebase sweep

on:
  schedule:
    - cron: '30 * * * *'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write
  actions: read

jobs:
  sweep:
    uses: sortie-ai/shared-workflows/.github/workflows/dependabot-rebase-sweep.yml@v1
    secrets:
      bot-token: ${{ secrets.CICDBOT_TOKEN }}
```

## Composite actions

### `dependabot-rebase`

Checks a Dependabot PR's merge state and requests a rebase if the branch is behind or conflicting.

```yaml
- uses: sortie-ai/shared-workflows/.github/actions/dependabot-rebase@v1
  with:
    pr-number: ${{ steps.pr.outputs.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    rebase-token: ${{ secrets.CICDBOT_TOKEN }}
```

| Output | Description |
|--------|-------------|
| `action` | `approve`, `rebase`, or `skip` |
| `reason` | Why (e.g. `ready`, `BEHIND`, `CONFLICTING`) |
| `merge-state` | Raw GitHub merge state value |

## Versioning

Pin to a **tag** in production:

```yaml
uses: sortie-ai/shared-workflows/.github/workflows/dependabot-merge.yml@v1
```

During development you may use `@main`, but never in production workflows.

## Required secrets

| Secret | Purpose | Used by |
|--------|---------|---------|
| `CICDBOT_TOKEN` | Bot PAT with push access | All workflows |

## License

MIT
