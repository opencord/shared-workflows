<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# GitHub Release Action

A composite GitHub Action that creates GitHub releases with artifacts using the `gh` CLI tool.

## Description

Builds release artifacts via `make`, generates SHA256 checksums, and creates a GitHub release with all artifacts attached. Supports draft releases for pre-release versions.

## Usage

```yaml
- uses: ./.github/actions/github-release-action
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `release-targets` | Make targets to build release artifacts | No | `release` |
| `artifact-glob` | Glob pattern for release artifacts | No | `release/*` |
| `github-token` | GitHub token for authentication | Yes | — |
| `draft` | Create a draft release | No | `false` |

### Example Workflow

```yaml
name: Release
on:
  push:
    tags:
      - "v*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: ./.github/actions/github-release-action
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          release-targets: "build package"
          artifact-glob: "dist/*"
```

## How It Works

1. Finds a SemVer tag on HEAD (`vX.Y.Z`, `X.Y.Z`, or `vX.Y.Z-devN`)
2. Builds release artifacts using `make <release-targets>`
3. Generates SHA256 checksums for all matched artifacts
4. Creates a GitHub release with artifacts and checksums attached
5. Marks the release as draft if the tag contains `-dev` or `draft` input is `true`

## Requirements

- Repository must have a SemVer tag on the current commit
- A `Makefile` with the specified release target(s)
- `gh` CLI (pre-installed on GitHub Actions runners)
- `GITHUB_TOKEN` with permissions to create releases
