<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# License Check Action

A GitHub Action that validates source code files contain proper license and copyright headers.

## Description

This composite action scans the repository for source code files and verifies that each file contains either a "Copyright" statement or "Apache License" reference in its header. This helps ensure compliance with open-source licensing requirements.

## Usage

```yaml
- name: Check License Headers
  uses: ./.github/actions/license-check-action
```

### Example Workflow

```yaml
name: License Compliance

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  license-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run license check
        uses: ./.github/actions/license-check-action
```

## How It Works

The action performs the following steps:

1. Recursively finds all files in the repository (excluding `.git` directory)
2. Filters files based on extension and name patterns
3. Searches each file for the text "Copyright" or "Apache License"
4. Reports any files missing the required license header
5. Fails the check if any violations are found

## Excluded File Types

The following file types and patterns are automatically excluded from license checking:

### Binary and Media Files
- Images: `.png`, `.jpg`, `.gif`, `.ico`, `.svg`, `.PNG`
- Fonts: `.eot`, `.ttf`, `.woff`
- Documents: `.pdf`, `.docx`, `.graffle`
- Archives: `.tar`, `.tar.gz`, `.jar`, `.oar`, `.csar`

### Configuration and Data Files
- `.json`, `.jsonld`, `.JSON`
- `.xml`, `.yaml`, `.yml`, `.toml`
- `.properties`, `.conf`, `.cfg`, `.cnf`, `.config`
- `.csv`, `.db`, `.log`
- `.txt`, `.md`, `.rst`

### Certificate and Key Files
- `.pem`, `.crt`, `.cert`, `.key`, `.csr`, `.der`
- `.jks`, `.p12`, `.asc`, `.gpg`

### Generated and Build Files
- `.pb.go`, `.pb.gw.go`, `*_pb2.py`, `*_pb2_grpc.py`
- `.pb.h`, `.pb.cc`
- `.pyc`, `.bin`
- `go.mod`, `go.sum`
- `.lock` files

### Special Files
- `Dockerfile`, `Dockerfile.*`
- `Makefile`, `Makefile.*`
- `README`
- `*ignore` files (e.g., `.gitignore`, `.dockerignore`)
- `*rc` files (e.g., `.bashrc`, `.npmrc`)

### Excluded Directories
- `vendor/`
- `conf/`
- `git/`
- `swagger/`
- `docs/`

## Troubleshooting

### False Positives

If files are incorrectly flagged as missing license headers:

1. Verify the file contains either "Copyright" or "Apache License" text
2. Check that the text appears in a comment format appropriate for the file type
3. Ensure there are no encoding issues preventing text detection
