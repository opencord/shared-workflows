<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# Docker Publish Action

A composite GitHub Action that builds and pushes Docker images tagged with branch names and git tags.

## Description

Builds Docker images using `make docker-build` and pushes them with `make docker-push`. Images are tagged with the current branch name and any git tags pointing at HEAD. Golang-style version tags (e.g., `v1.2.3`) have the leading `v` stripped automatically.

## Usage

```yaml
- uses: ./.github/actions/docker-publish-action
  with:
    docker-registry: "docker.io"
    docker-repository: "myorg"
    image-name: "myapp"
    docker-username: ${{ secrets.DOCKER_USERNAME }}
    docker-password: ${{ secrets.DOCKER_PASSWORD }}
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `docker-registry` | Yes | — | Docker registry URL |
| `docker-repository` | Yes | — | Docker repository prefix |
| `image-name` | Yes | — | Name of the Docker image |
| `extra-environment-vars` | No | `""` | Extra environment variables for make commands |
| `docker-username` | Yes | — | Docker registry username |
| `docker-password` | Yes | — | Docker registry password |

### Example Workflow

```yaml
name: Publish Docker Image

on:
  push:
    branches: [main]
    tags: ["v*"]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: ./.github/actions/docker-publish-action
        with:
          docker-registry: "docker.io"
          docker-repository: "voltha"
          image-name: "voltha-northbound"
          extra-environment-vars: "GOFLAGS=-buildvcs=false"
          docker-username: ${{ secrets.DOCKER_USERNAME }}
          docker-password: ${{ secrets.DOCKER_PASSWORD }}
```

## How It Works

1. Logs in to the configured Docker registry.
2. Discovers git tags pointing at the current HEAD commit.
3. Runs `make docker-build` with `DOCKER_TAG` set to the branch name.
4. Runs `make docker-build` for each git tag (stripping the `v` prefix).
5. Runs `make docker-push` for the branch tag and each git tag.
6. Logs out of the Docker registry.

## Requirements

- The repository must have a `Makefile` with `docker-build` and `docker-push` targets.
- These targets must respect `DOCKER_TAG`, `DOCKER_REGISTRY`, `DOCKER_REPOSITORY`, and `IMAGE_NAME` environment variables.
- The checkout step should use `fetch-depth: 0` to ensure git tags are available.
