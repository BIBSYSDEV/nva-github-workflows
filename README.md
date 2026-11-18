# NVA Github workflows

This repository holds GitHub workflows that are intended to be shared across NVA repositories.

## Development

Versioning is done by keeping one long-lived branch per major version that can be referenced by calling workflows in other repositories.
Do not introduce breaking changes in existing branches.
To add changes that would require changes in caller workflows, create a new branch from the current newest major version with the same name pattern (e.g. `v2`, `v3` etc).

## Usage

### v2 (recommended)

```yml
# Filename: .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, reopened, synchronize]

permissions:
  contents: read
  checks: write
  pull-requests: write

jobs:
  build:
    uses: BIBSYSDEV/nva-github-workflows/.github/workflows/java.yml@v2
    secrets:
      codacy_token: ${{ secrets.CODACY_PROJECT_TOKEN }}
```

This workflow does not lint OpenAPI documents.
Lint them with [Spectral](https://github.com/stoplightio/spectral) in the Gradle build instead, through `nva.spectral` in [nva-gradle-plugins](https://github.com/BIBSYSDEV/nva-gradle-plugins#configuration):

```groovy
nva {
    spectral {
        documents = ['docs/*.yaml']
        rulesetFile = file('.spectral.yaml') // optional, defaults to the ruleset bundled with nva-gradle-plugins
    }
}
```

The inputs `lint_openapi` and `openapi_files` are deprecated and ignored.
They stay declared because a caller that passes an undeclared input fails, and will be removed once no caller passes them.

The CloudFormation linting job runs [cfn-lint](https://github.com/aws-cloudformation/cfn-lint) on `template.yaml` in the repository root.
To lint other templates, list them under `templates` in a `.cfnlintrc.yaml` in the repository root:

```yaml
templates:
  - infrastructure/*.yaml
```

Running `cfn-lint` locally in the repository root picks up the same file.
Set `lint_cloudformation: false` to skip the job.

### v1 (legacy)

```yml
# Filename: .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, reopened, synchronize]

permissions:
  contents: read
  checks: write
  pull-requests: write

jobs:
  build:
    uses: BIBSYSDEV/nva-github-workflows/.github/workflows/java.yml@v1
    secrets:
      codacy_token: ${{ secrets.CODACY_PROJECT_TOKEN }}
```

## Dependabot

Every NVA repository should have a Dependabot config that follows the shared [template](templates/dependabot.yml).
Copy it to `.github/dependabot.yml` and change only what the repository needs.
The template is the source of truth, so change it here first when the convention changes.

What it sets up:

- Weekly runs on Monday at 07:00, Europe/Oslo.
- One grouped pull request per ecosystem for minor and patch updates.
  Major updates get a pull request each.
- A 7-day cooldown, so a release that gets yanked or patched is usually caught before it reaches us.
  Our own artifacts (`com.github.bibsysdev:*`) are exempt, since we release them ourselves, usually to get a fix out.
- Commit message prefixes `build` for dependencies and `ci` for GitHub Actions.

The template is written for Gradle repositories.
Listing the root directory is enough for Gradle: Dependabot has been observed to open pull requests for `buildSrc/build.gradle` and `build-logic/build.gradle` with only `/` configured.

For an npm or uv repository, change `package-ecosystem` to `npm` or `uv`, rename the group to match (`npm-minor-and-patch`, `python-minor-and-patch`), and remove the cooldown `exclude`.
A repository with more than one ecosystem gets one `updates` entry per ecosystem.

Dependabot does not follow `nvaCommonsVersion` in `gradle.properties`, so nva-commons and the shared version catalog are still bumped by hand.
