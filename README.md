# renovate-config

Shared Renovate preset and reusable GitHub Actions workflows for Node.js / pnpm repositories.

## Private repository access

This repository is private. Reuse only works after:

1. **GitHub Actions** — in this repository, set **Settings → Actions → General → Access** to **Accessible from repositories owned by the user**.
2. **Renovate** — grant the Renovate GitHub App access to this repository so it can read the preset.

Consumer repositories must also have the Renovate app installed (or otherwise run Renovate with a token that can read this repo).

## Renovate preset

Product repos:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>tessOu56/renovate-config"]
}
```

Labs repos (same grouping and Node/pnpm constraints, no automerge):

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>tessOu56/renovate-config:labs"]
}
```

### Policy

| Update | PR shape | Automerge |
| --- | --- | --- |
| Patch, pin, digest | As produced by Renovate | Yes, after required CI passes |
| Security / vulnerability fixes | Immediate; `minimumReleaseAge` does not apply | Yes in the default preset (not labs), after required CI passes |
| Minor | One grouped PR each Monday (`Asia/Taipei`) | No |
| Major | One package per PR | Never; review required |

Other rules:

- `minimumReleaseAge` is 3 days (vulnerability alerts may bypass this).
- Every PR is labeled `deps` and uses the `renovate/` branch prefix.
- GitHub Actions are pinned to commit SHAs with version comments.
- Node stays on 24.x (`>=24.22.1 <25`). Align `.nvmrc`, `engines.node`, and `packageManager: pnpm@11.x`. Node majors beyond 24 are not proposed automatically.
- pnpm and `packageManager` updates are grouped and kept on 11.x.
- Lock-file maintenance runs monthly.

Create a `deps` label in each consumer repository if it does not already exist.

## Reusable workflows

Pin the reusable workflow to a commit SHA or tag of this repository (not a moving branch):

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    uses: tessOu56/renovate-config/.github/workflows/ci-node.yml@<sha-or-tag>
    with:
      command: pnpm test
      audit: true
      audit-level: high
      # working-directory: .   # optional; default is the repo root

  deps-track:
    uses: tessOu56/renovate-config/.github/workflows/deps-track-guard.yml@<sha-or-tag>
```

`ci-node.yml` checks out the caller, installs pnpm from `package.json` `packageManager`, sets up Node from `.nvmrc` with the pnpm cache, then runs `pnpm install --frozen-lockfile`, optional `pnpm audit`, and the smoke/test command.

`deps-track-guard.yml` fails pull requests that change existing dependency versions in `package.json`, `packageManager`, `engines.node`, or `.nvmrc` unless the branch is `renovate/*` or `chore/deps-*`. Adding or removing packages on a feature branch is allowed; bumping versions is not.

Do not call these workflows from `pull_request_target`.
