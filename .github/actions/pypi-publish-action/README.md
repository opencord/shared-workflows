<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# PyPI Publish Action

A composite GitHub Action that builds and publishes Python packages to PyPI on SemVer release tags.

## Description

Publishes Python packages to PyPI when the current HEAD has a valid SemVer release tag (vX.Y.Z or X.Y.Z). Supports publishing multiple modules from a single repository using pipe-separated directory paths.

## Usage

```yaml
- uses: ./.github/actions/pypi-publish-action
  with:
    pypi-username: ${{ secrets.PYPI_USERNAME }}
    pypi-password: ${{ secrets.PYPI_PASSWORD }}
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `pypi-index` | No | `pypi` | PyPI repository name to publish to |
| `pypi-username` | Yes | — | PyPI username |
| `pypi-password` | Yes | — | PyPI password or API token |
| `module-dirs` | No | `.` | Pipe-separated list of directories containing Python modules to publish |
| `prep-commands` | No | `""` | Commands to run before building |

### Example Workflow

```yaml
name: Publish to PyPI

on:
  push:
    tags: ["v*"]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - uses: ./.github/actions/pypi-publish-action
        with:
          pypi-username: ${{ secrets.PYPI_USERNAME }}
          pypi-password: ${{ secrets.PYPI_PASSWORD }}
          module-dirs: "lib/core|lib/utils"
          prep-commands: "make generate"
```

## How It Works

1. Gets the git tag on HEAD and verifies it is a SemVer release version (vX.Y.Z or X.Y.Z).
2. Exits gracefully if no valid release tag is found.
3. Runs any specified prep commands.
4. For each module directory: verifies `setup.py` exists, installs build dependencies, builds with `python3 -m build`, and uploads with `twine upload`.

## Requirements

- Each module directory must contain a `setup.py`.
- The checkout step should use `fetch-depth: 0` to ensure git tags are available.
- Python must be available in the runner environment.
