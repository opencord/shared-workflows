<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# Helm Lint Action

A GitHub Action that lints and tests Helm charts using the chart-testing tool.

## Description

This composite action validates Helm charts by linting their structure and installing them on a kind cluster. It only processes charts that have changed compared to the target branch.

## Usage

```yaml
- name: Lint and Test Charts
  uses: ./.github/actions/helm-lint-action
  with:
    target-branch: main
```

### Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `target-branch` | Target branch to compare against for detecting changed charts | Yes |

### Example Workflow

```yaml
name: Helm Chart CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - name: Lint and test Helm charts
        uses: ./.github/actions/helm-lint-action
        with:
          target-branch: main
```

## How It Works

The action performs the following steps:

1. Checks out the repository with full git history
2. Sets up Helm and Python (required for chart-testing)
3. Installs the chart-testing CLI tool
4. Detects charts that changed compared to the target branch
5. Lints the changed charts
6. Creates a kind Kubernetes cluster
7. Installs and tests the changed charts on the cluster

## Requirements

- Repository must contain valid Helm charts
- Charts should follow Helm best practices for linting to pass

## Configuration

For advanced configuration, add a `ct.yaml` file to your repository root. See the [chart-testing documentation](https://github.com/helm/chart-testing) for available options.

## Dependencies

This action uses the following tools:

- [helm/chart-testing-action](https://github.com/helm/chart-testing-action) - Chart testing CLI
- [azure/setup-helm](https://github.com/azure/setup-helm) - Helm installation
- [helm/kind-action](https://github.com/helm/kind-action) - Kind cluster creation
