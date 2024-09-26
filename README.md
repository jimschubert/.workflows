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
      golangci-lint: "v1.61"
    secrets: inherit
```
