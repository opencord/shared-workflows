<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# ShellCheck Action

A GitHub Action that lints shell scripts using ShellCheck.

## Description

This composite action finds all shell scripts (`.sh` and `.bash` files) in the specified path and validates them using ShellCheck. It reports any issues and fails if linting errors are found.

## Usage

```yaml
- name: Lint Shell Scripts
  uses: ./.github/actions/shellcheck-action
```

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to search for shell scripts | No | `.` |
| `severity` | Minimum severity level (error, warning, info, style) | No | `style` |
| `exclude` | Comma-separated list of ShellCheck codes to exclude | No | `""` |

### Example Workflow

```yaml
name: Shell Script Lint

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  shellcheck:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Lint shell scripts
        uses: ./.github/actions/shellcheck-action
        with:
          path: scripts
          severity: warning
          exclude: SC1090,SC2034
```

## How It Works

The action performs the following steps:

1. Installs ShellCheck if not already present
2. Finds all `.sh` and `.bash` files in the specified path
3. Runs ShellCheck on each file with the configured severity level
4. Reports pass/fail status for each file
5. Fails the action if any script has linting errors

## Severity Levels

- `error` - Only show errors
- `warning` - Show errors and warnings
- `info` - Show errors, warnings, and informational messages
- `style` - Show all issues including style suggestions (default)

## Requirements

- Ubuntu runner (ShellCheck is installed via apt-get)
- Repository checkout step should precede this action
