<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# Tag Release Action

A composite GitHub Action that creates and pushes a release git tag from the
version in the repository's `VERSION` file.

## Description

Intended to run on every merge to the default branch. It reads the `VERSION`
file and, unless the version is a development version (ends with `-dev` or
`-devN`), creates an annotated git tag for it and pushes it to `origin`.
Downstream release steps can then be gated on the `release` output.

The tag naming convention matches `tag-check-action`: Go projects (those with
a `go.mod` or `Gopkg.toml`) get a `v` prefix (`v1.2.3`); all other projects
are tagged with the bare version (`1.2.3`).

Because the tag is pushed from within the same workflow run that publishes the
release, no personal access token is required: tags pushed with the default
`GITHUB_TOKEN` do not trigger further workflow runs, which is exactly the
behaviour wanted here.

## Usage

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- id: tag
  uses: opencord/shared-workflows/.github/actions/tag-release-action@main
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `path` | No | `.` | Path to the repository to tag |
| `version-file` | No | `VERSION` | File (relative to `path`) containing the version |
| `tag-prefix` | No | `auto` | Tag prefix. `auto` uses `v` for Go projects and nothing otherwise; any other value is used literally (use `""` for none) |
| `git-user-name` | No | `github-actions[bot]` | Name recorded as the tagger of the annotated tag |
| `git-user-email` | No | `41898282+github-actions[bot]@users.noreply.github.com` | Email recorded as the tagger of the annotated tag |
| `dry-run` | No | `false` | Compute outputs but do not create or push the tag |

### Outputs

| Output | Description |
|--------|-------------|
| `version` | Version read from the version file |
| `tag` | Git tag derived from the version (empty for `-dev` versions) |
| `created` | `true` if the tag was created and pushed by this run |
| `release` | `true` if HEAD now carries the release tag (newly created or pre-existing), i.e. release artifacts should be published |

### Example Workflow

```yaml
name: release

on:
  workflow_dispatch:
  push:
    branches:
      - master
      - main

permissions:
  contents: write

jobs:
  tag:
    runs-on: ubuntu-latest
    outputs:
      release: ${{ steps.tag.outputs.release }}
      tag: ${{ steps.tag.outputs.tag }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - id: tag
        uses: opencord/shared-workflows/.github/actions/tag-release-action@main

  release:
    needs: tag
    if: needs.tag.outputs.release == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: opencord/shared-workflows/.github/actions/docker-publish-action@main
        with:
          docker-registry: ""
          docker-repository: "voltha/"
          image-name: "myapp"
          branch-tag: ${{ github.ref_name }}
          docker-username: ${{ secrets.DOCKER_USERNAME }}
          docker-password: ${{ secrets.DOCKER_PASSWORD }}
```

## How It Works

1. Reads the first non-blank line of the version file
2. If the version ends with `-dev` (or `-devN`), exits successfully with
   `release=false`
3. Derives the tag name (`v`-prefixed for Go projects)
4. Looks the tag up on `origin`:
   - Not present: creates an annotated tag at HEAD, pushes it, and sets
     `created=true` and `release=true`
   - Present and pointing at HEAD (e.g. a manual re-run): sets `release=true`
   - Present but pointing elsewhere (version not bumped): emits a warning and
     sets `release=false`

## Requirements

- The checkout step must have `fetch-depth: 0` and push credentials (the
  default `actions/checkout` behaviour with `persist-credentials: true`)
- The workflow needs `permissions: contents: write`
- The repository must contain the version file (`VERSION` by default)
