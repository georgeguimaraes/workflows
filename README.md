# Shared GitHub Workflows

Reusable workflows and composite actions for my repositories.

Pin each reference to the commit SHA of a release tag, with the tag in a comment. The repositories that use these workflows require SHA pins, and Dependabot reads the comment to send update PRs.

## Reusable Workflows

### release-please.yml

Runs [release-please](https://github.com/googleapis/release-please) to create release PRs based on conventional commits.

```yaml
jobs:
  release-please:
    uses: georgeguimaraes/workflows/.github/workflows/release-please.yml@0895359397b7e82725d0dce6d5fc2af8068a6ea1 # v1.0.1
```

Expects `release-please-config.json` and `.release-please-manifest.json` in your repo.

### on-release-published.yml

Checks out a release tag and publishes the package to Hex.pm.

```yaml
jobs:
  hex-publish:
    uses: georgeguimaraes/workflows/.github/workflows/on-release-published.yml@0895359397b7e82725d0dce6d5fc2af8068a6ea1 # v1.0.1
    with:
      tag: v1.2.3            # required
      elixir-version: "1.18" # optional, default: 1.18
      otp-version: "27"      # optional, default: 27
    secrets:
      hex-api-key: ${{ secrets.HEX_API_KEY }}
```

## Composite Actions

### elixir-setup

Sets up Elixir/OTP, restores dependency cache, and runs `mix deps.get`.

```yaml
- uses: georgeguimaraes/workflows/actions/elixir-setup@0895359397b7e82725d0dce6d5fc2af8068a6ea1 # v1.0.1
  with:
    otp-version: "27"      # optional, default: 27
    elixir-version: "1.18" # optional, default: 1.18
```

### hex-publish

Publishes an Elixir package to Hex.pm.

```yaml
- uses: georgeguimaraes/workflows/actions/hex-publish@0895359397b7e82725d0dce6d5fc2af8068a6ea1 # v1.0.1
  with:
    hex-api-key: ${{ secrets.HEX_API_KEY }}
```

## Example: Elixir Release Workflow

release-please opens a release PR. When you merge it, release-please creates the release, and the next job publishes it to Hex.pm.

```yaml
name: Release Please

on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@45996ed1f6d02564a971a2fa1b5860e934307cf7 # v5.0.0
        id: release

  hex-publish:
    needs: release-please
    if: ${{ needs.release-please.outputs.release_created == 'true' }}
    uses: georgeguimaraes/workflows/.github/workflows/on-release-published.yml@0895359397b7e82725d0dce6d5fc2af8068a6ea1 # v1.0.1
    with:
      tag: ${{ needs.release-please.outputs.tag_name }}
    secrets:
      hex-api-key: ${{ secrets.HEX_API_KEY }}
```
