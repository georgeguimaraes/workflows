# Shared GitHub Workflows

Reusable workflows and composite actions for my repositories.

## Reusable Workflows

### release-please.yml

Runs [release-please](https://github.com/googleapis/release-please) to create release PRs based on conventional commits.

```yaml
jobs:
  release-please:
    uses: georgeguimaraes/workflows/.github/workflows/release-please.yml@main
```

Expects `release-please-config.json` and `.release-please-manifest.json` in your repo.

## Composite Actions

### elixir-setup

Sets up Elixir/OTP, restores dependency cache, and runs `mix deps.get`.

```yaml
- uses: georgeguimaraes/workflows/actions/elixir-setup@main
  with:
    otp-version: "27"      # optional, default: 27
    elixir-version: "1.18" # optional, default: 1.18
```

### create-release

Creates a git tag and GitHub release with auto-generated notes. Language-agnostic.

```yaml
- uses: georgeguimaraes/workflows/actions/create-release@main
  with:
    version: "1.2.3" # required, without v prefix
```

### extract-version

Extracts version from a release PR title or workflow input.

```yaml
- uses: georgeguimaraes/workflows/actions/extract-version@main
  id: version
  with:
    version: ${{ inputs.version }}           # optional, from workflow_dispatch
    pr-title: ${{ github.event.pull_request.title }} # optional

- run: echo "Version is ${{ steps.version.outputs.version }}"
```

### mark-release-tagged

Updates a release PR label from `autorelease: pending` to `autorelease: tagged`.

```yaml
- uses: georgeguimaraes/workflows/actions/mark-release-tagged@main
  with:
    pr-number: ${{ github.event.pull_request.number }}
```

### hex-publish

Publishes an Elixir package to Hex.pm.

```yaml
- uses: georgeguimaraes/workflows/actions/hex-publish@main
  with:
    hex-api-key: ${{ secrets.HEX_API_KEY }}
```

## Example: Elixir Release Workflow

```yaml
name: Create Release

on:
  pull_request:
    types: [closed]

jobs:
  release:
    if: github.event.pull_request.merged && contains(github.event.pull_request.labels.*.name, 'autorelease: pending')
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v6

      - uses: georgeguimaraes/workflows/actions/extract-version@main
        id: version
        with:
          pr-title: ${{ github.event.pull_request.title }}

      - uses: georgeguimaraes/workflows/actions/elixir-setup@main

      - run: mix test

      - uses: georgeguimaraes/workflows/actions/create-release@main
        with:
          version: ${{ steps.version.outputs.version }}

      - uses: georgeguimaraes/workflows/actions/hex-publish@main
        with:
          hex-api-key: ${{ secrets.HEX_API_KEY }}
```
