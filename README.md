# jimschubert/.workflows

These are my own [shared workflows](https://github.blog/developer-skills/github/using-reusable-workflows-github-actions/) to simplify my workflow setup.

Each workflow may define some inputs which may be overridden by the caller workflow.

Example usage:

```yaml
name: Build Go
on:
  push:
    branches: [ 'main', 'feature/*' ]
  pull_request:

jobs:
  build:
    uses: jimschubert/.workflows/.github/workflows/go-build.yml@main
    with:
      golangci-lint-version: "v2.7.2"
    secrets: inherit
```

## Workflows

| Name | Description |
| --- | --- |
| `codeql.yml` | Runs GitHub CodeQL security analysis. Supports multiple languages. |
| `go-build.yml` | Builds and tests Go code, uploads coverage. |
| `go-build-goreleaser.yml` | Runs `go-build.yml` and publishes via GoReleaser. |
| `go-lint.yml` | Runs `golangci-lint`. |
| `go-pr.yml` | Dedicated pull request build with coverage. |
| `go-release.yml` | Generates changelog and creates GitHub release. |
| `tag-next-version.yml` | Bumps semantic version and creates a tag. Optionally creates a release. |
