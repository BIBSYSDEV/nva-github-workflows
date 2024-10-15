# NVA Github workflows

This repository holds GitHub workflows that are intended to be shared across NVA repositories.

## Development

Versioning is done by keeping one long-lived branch per major version that can be referenced by calling workflows in other repositories.
Do not introduce breaking changes in existing branches.
To add changes that would require changes in caller workflows, create a new branch from the current newest major version with the same name pattern (e.g. `v2`, `v3` etc).

## Usage

Example caller workflow:

```yml
# Filename: .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  build:
    uses: BIBSYSDEV/nva-github-workflows/.github/workflows/java.yml@v1
    secrets:
      codacy_token: ${{ secrets.CODACY_PROJECT_TOKEN }}
```
