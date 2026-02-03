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
| `go-build.yml` | Builds and tests Go code, uploads coverage. |
| `go-build-goreleaser.yml` | Runs `go-build.yml` and publishes via GoReleaser. Supports automatic DHI Docker registry login when Dockerfile(s) contain `dhi.io`. |
| `go-lint.yml` | Runs `golangci-lint`. |
| `go-pr.yml` | Dedicated pull request build with coverage. |
| `go-release.yml` | Generates changelog and creates GitHub release. |

### Docker Registry Support

The `go-build-goreleaser.yml` workflow automatically detects and logs into Docker registries based on your repository's Dockerfiles:

- **Docker Hub**: Enabled when `DOCKER_USERNAME` and `DOCKER_PASSWORD` secrets are set
- **DHI Registry** (`dhi.io`): Automatically enabled when:
  - `DHI_USERNAME` and `DHI_PASSWORD` secrets are set, AND
  - Any `Dockerfile` or `*.Dockerfile` in the repository root contains `dhi.io`

**Required Secrets:**
- `DOCKER_USERNAME`, `DOCKER_PASSWORD` - Docker Hub credentials
- `DHI_USERNAME`, `DHI_PASSWORD` - DHI registry credentials (optional, only needed if using dhi.io)
