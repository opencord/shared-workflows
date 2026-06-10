<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# Maven Publish Action

Build and publish Maven (Java) artifacts to a remote repository.

## Description

A composite GitHub Action that sets up JDK, optionally imports a GPG signing
key, configures Maven authentication, and executes Maven goals to publish
artifacts. Based on Jenkins JJB Maven release patterns.

## Usage

```yaml
- uses: ./.github/actions/maven-publish-action
  with:
    server-username: ${{ secrets.OSSRH_USERNAME }}
    server-password: ${{ secrets.OSSRH_PASSWORD }}
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | No | `11` | Java version to use |
| `java-distribution` | No | `temurin` | Java distribution |
| `maven-goals` | No | `-Prelease clean deploy` | Maven goals to execute |
| `maven-settings-file` | No | `""` | Path to custom Maven settings.xml |
| `gpg-private-key` | No | `""` | GPG private key for signing |
| `gpg-passphrase` | No | `""` | GPG passphrase |
| `server-id` | No | `ossrh` | Maven server ID for deployment |
| `server-username` | **Yes** | — | Maven server username |
| `server-password` | **Yes** | — | Maven server password |

### Example Workflow

```yaml
name: Maven Publish
on:
  push:
    tags:
      - "v*"

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ./.github/actions/maven-publish-action
        with:
          java-version: "17"
          server-username: ${{ secrets.OSSRH_USERNAME }}
          server-password: ${{ secrets.OSSRH_PASSWORD }}
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

## How It Works

1. Sets up JDK using `actions/setup-java@v4` with the specified distribution and version
2. Imports the GPG private key for artifact signing (if provided)
3. Configures Maven settings with server credentials (custom or generated)
4. Executes Maven with the specified goals using `MAVEN_USERNAME` and `MAVEN_PASSWORD` environment variables for authentication

## Requirements

- Repository must contain a valid `pom.xml`
- Server credentials must be provided (typically via GitHub Secrets)
- GPG key required only if the Maven profile performs artifact signing
