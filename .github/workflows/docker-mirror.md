# Docker Mirror Workflow

Mirrors multi-platform Docker images from one registry to another, preserving all architectures and attestations.

## Features

- ✅ Mirrors all architectures (amd64, arm64, etc.)
- ✅ Preserves attestations (signatures, SBOMs)
- ✅ Supports custom tagging strategies
- ✅ Automatic major/minor version tag generation
- ✅ Configurable registry authentication

## Usage

### Basic Example

```yaml
jobs:
  mirror:
    uses: jimschubert/.workflows/.github/workflows/docker-mirror.yml@main
    with:
      source-image: docker.io/myorg/myapp:v1.2.3
      target-image: ghcr.io/myorg/myapp
    secrets: inherit
```

### Full Example with Version Tags

```yaml
jobs:
  mirror-to-ghcr:
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    uses: jimschubert/.workflows/.github/workflows/docker-mirror.yml@main
    with:
      source-image: jimschubert/labeler:${{ github.ref_name }}
      target-image: ghcr.io/jimschubert/labeler/labeler
      target-registry: ghcr.io
      create-major-tag: true      # Creates v1
      create-minor-tag: true      # Creates v1.2
      create-latest-tag: true     # Creates latest
    secrets: inherit
```

### Custom Registry with Authentication

```yaml
jobs:
  mirror:
    uses: jimschubert/.workflows/.github/workflows/docker-mirror.yml@main
    with:
      source-image: docker.io/myorg/myapp:v1.2.3
      target-image: registry.example.com/myorg/myapp
      target-registry: registry.example.com
      target-username: myuser
      additional-tags: stable,production
    secrets:
      target-password: ${{ secrets.REGISTRY_TOKEN }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `source-image` | Yes | - | Source Docker image with tag (e.g., `jimschubert/labeler:v1.0.0`) |
| `target-image` | Yes | - | Target Docker image without tag (e.g., `ghcr.io/jimschubert/labeler`) |
| `target-registry` | No | `ghcr.io` | Target registry URL |
| `target-username` | No | `github.actor` | Username for target registry authentication |
| `additional-tags` | No | `''` | Comma-separated list of additional tags (e.g., `"stable,production"`) |
| `create-major-tag` | No | `false` | Create major version tag (v1) from source semantic version |
| `create-minor-tag` | No | `false` | Create minor version tag (v1.2) from source semantic version |
| `create-latest-tag` | No | `false` | Create `latest` tag |

## Secrets

| Secret | Required | Default | Description |
|--------|----------|---------|-------------|
| `target-password` | No | `secrets.GITHUB_TOKEN` | Password/token for target registry authentication |

## Permissions Required

```yaml
permissions:
  contents: read
  packages: write  # Required when mirroring to GHCR
```

## How It Works

1. **Authenticates** to the target registry using provided credentials
2. **Sets up Docker Buildx** for multi-platform operations
3. **Extracts version** information from source tag (if version tags are requested)
4. **Inspects source image** to verify all platforms and attestations
5. **Creates manifest list** on target registry pointing to all platform images
6. **Verifies** the mirror was successful

## Example Output

```
=== Source image platforms ===
Name:      docker.io/jimschubert/labeler:v1.0.0
MediaType: application/vnd.oci.image.index.v1+json
Digest:    sha256:abc123...

Manifests:
  Platform: linux/amd64
  Platform: linux/arm64
  Attestations: 2

=== Creating the following tags ===
  ghcr.io/jimschubert/labeler:v1.0.0
  ghcr.io/jimschubert/labeler:v1
  ghcr.io/jimschubert/labeler:v1.0
  ghcr.io/jimschubert/labeler:latest

=== Mirroring multi-platform image ===
✅ Successfully mirrored all platforms and attestations to target registry
```

## Notes

- The workflow uses `docker buildx imagetools` which operates on manifest lists
- No actual image layers are pulled to the runner - this is a registry-to-registry operation
- All platforms and attestations are preserved automatically
- The source tag is always created on the target (e.g., `v1.2.3` → `v1.2.3`)
- Version extraction works with semantic versioning tags (with or without 'v' prefix)
