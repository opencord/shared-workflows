<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# Tag Check Action

A GitHub Action that validates version tags and Dockerfile parent images.

## Description

This composite action validates that repository versions follow SemVer conventions, checks for duplicate git tags, and ensures Dockerfile parent images specify proper versions.

## Usage

```yaml
- name: Validate Tags
  uses: ./.github/actions/tag-check-action
```

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `semver-strict` | Require strict SemVer versions | No | `false` |
| `dockerparent-strict` | Require versioned parent images in Dockerfiles | No | `true` |
| `path` | Path to the repository to check | No | `.` |

### Example Workflow

```yaml
name: Version Validation

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  tag-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Validate version tags
        uses: ./.github/actions/tag-check-action
        with:
          semver-strict: true
```

## How It Works

The action performs the following steps:

1. Reads version from `VERSION`, `package.json`, or `pom.xml`
2. Validates version sequence (parent version must exist)
3. Checks if version is a valid SemVer release
4. For release versions:
   - Verifies the tag doesn't already exist
   - Validates Dockerfile parent images have proper versions

## Version File Detection

The action looks for versions in this order:

1. `VERSION` file (first line)
2. `package.json` (version field)
3. `pom.xml` (project version)

For Go projects (`go.mod` present), tags are prefixed with `v`.

## Dockerfile Validation

Parent images are validated to ensure they use:

- SemVer versions (e.g., `1.2.3`)
- SHA256 hashes (e.g., `@sha256:abc123...`)
- Major.Minor versions (e.g., `16.04`, `3.8-alpine`)
- Special cases: `scratch`, `gcr.io/distroless/static:nonroot`

## Requirements

- Repository checkout with full history (`fetch-depth: 0`)
- Ubuntu runner (xmllint is installed via apt-get if needed)
