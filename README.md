# Reusable GitHub Actions Workflows

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Actions](https://img.shields.io/badge/GitHub-Actions-blue.svg)](https://github.com/features/actions)
[![actionlint](https://img.shields.io/badge/validated-actionlint-green.svg)](https://github.com/rhysd/actionlint)

Production-ready, reusable GitHub Actions workflows for modern software development. Built with the Single Responsibility Principle, these workflows provide composable building blocks for CI/CD pipelines across your organization.

## Features

- 🔧 **Modular Design** - Each workflow has a single, well-defined responsibility
- 🛡️ **Security First** - SBOM, provenance attestation, and comprehensive security scanning
- 📦 **Production Ready** - Battle-tested with explicit version pinning and error handling
- 🚀 **Easy to Use** - Copy templates and start using in minutes
- 🔄 **Composable** - Mix and match workflows to build custom pipelines
- ✅ **Well Tested** - All workflows validated with actionlint

## Quick Start

### 1. Simple Go Project

```yaml
# .github/workflows/pr.yml
name: Pull Request

on: pull_request

jobs:
  go-ci:
    uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@main
    with:
      go-version: "1.25.5"
      coverage-threshold: 80
```

### 2. Go Service with Docker

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  build:
    uses: mandacode-lab/workflows/.github/workflows/docker-build.yml@main
    with:
      images: '[{"name": "api", "dockerfile": "Dockerfile"}]'
      registry: ghcr.io
      repository: ${{ github.repository }}
      tag: ${{ github.ref_name }}
      tag-prefix: "v"
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Full Stack with Kubernetes

```yaml
jobs:
  docker-build:
    uses: mandacode-lab/workflows/.github/workflows/docker-build.yml@main
    # ... docker configuration

  helm-package:
    needs: docker-build
    uses: mandacode-lab/workflows/.github/workflows/helm-package.yml@main
    # ... helm configuration
```

## Available Workflows

### Docker Workflows

| Workflow | Description | Use Case |
|----------|-------------|----------|
| [docker-test.yml](.github/workflows/docker-test.yml) | Dockerfile linting and security scanning | PR validation |
| [docker-build.yml](.github/workflows/docker-build.yml) | Build and push to OCI registries | GHCR, Docker Hub, Harbor |
| [docker-build-ecr.yml](.github/workflows/docker-build-ecr.yml) | Build and push to AWS ECR | AWS deployments |

**Key Features:**
- Hadolint v3.0.0 for Dockerfile best practices
- Trivy 0.28.0 for vulnerability scanning
- Multi-platform builds (linux/amd64, linux/arm64)
- SBOM and provenance attestation
- Automatic repository creation (ECR)

### Go Workflows

| Workflow | Description |
|----------|-------------|
| [go-ci.yml](.github/workflows/go-ci.yml) | Complete Go CI pipeline |

**Includes:**
- Build verification
- Test execution with coverage reporting
- golangci-lint v2.2.0
- Security scanning (gosec 2.21.4, govulncheck)
- Optional Codecov integration

### Helm Workflows

| Workflow | Description |
|----------|-------------|
| [helm-test.yml](.github/workflows/helm-test.yml) | Comprehensive Helm chart testing |
| [helm-package.yml](.github/workflows/helm-package.yml) | Package and publish to OCI registries |

**Features:**
- Helm lint and template validation
- kubeconform v0.24.0 for Kubernetes schema validation
- Kind-based installation testing
- Library chart support
- Optional GPG signing

### Validation

| Workflow | Description |
|----------|-------------|
| [workflow-lint.yml](.github/workflows/workflow-lint.yml) | Workflow validation and linting |

**Validates:**
- GitHub Actions syntax (actionlint)
- YAML formatting (yamllint)
- Deprecated syntax detection
- Workflow structure

## Templates

Pre-built templates for common scenarios are available in the [`templates/`](templates/) directory:

- **PR Workflows**
  - `go-simple-pr.yml` - Go only
  - `go-docker-pr.yml` - Go + Docker
  - `go-docker-helm-pr.yml` - Full stack

- **Release Workflows**
  - `go-docker-release-ghcr.yml` - GHCR deployment
  - `go-docker-release-ecr.yml` - AWS ECR deployment
  - `go-docker-helm-release.yml` - Kubernetes deployment
  - `multi-image-docker.yml` - Microservices

### Using Templates

```bash
# Copy template to your project
cp templates/go-docker-pr.yml your-project/.github/workflows/pr.yml

# Customize for your project
# - Update repository references
# - Adjust versions and paths
# - Configure coverage thresholds
```

See the [templates README](templates/README.md) for detailed instructions.

## Documentation

### Docker Build Configuration

<details>
<summary>Click to expand</summary>

**Multi-image builds:**
```yaml
with:
  images: |
    [
      {"name": "api", "dockerfile": "Dockerfile"},
      {"name": "worker", "dockerfile": "services/worker/Dockerfile"},
      {"name": "migration", "dockerfile": "docker/migration/Dockerfile"}
    ]
```

**Custom build arguments:**
```yaml
with:
  build-args: |
    VERSION=${{ github.ref_name }}
    BUILD_DATE=${{ github.event.head_commit.timestamp }}
    GIT_COMMIT=${{ github.sha }}
```

**Multi-platform builds:**
```yaml
with:
  platforms: "linux/amd64,linux/arm64,linux/arm/v7"
```

</details>

### AWS ECR Authentication

<details>
<summary>Click to expand</summary>

**OIDC (Recommended):**
```yaml
permissions:
  id-token: write
  contents: read

jobs:
  build:
    uses: mandacode-lab/workflows/.github/workflows/docker-build-ecr.yml@main
    secrets:
      aws-role-arn: arn:aws:iam::123456789012:role/GithubActionsRole
```

**Access Keys:**
```yaml
secrets:
  aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
  aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

</details>

### Helm Chart Configuration

<details>
<summary>Click to expand</summary>

**Multiple charts:**
```yaml
with:
  charts: |
    [
      {"dir": "charts/api"},
      {"dir": "charts/worker"},
      {"dir": "charts/frontend"}
    ]
```

**With signing:**
```yaml
with:
  enable-signing: true
secrets:
  gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
  gpg-key-name: "your.email@example.com"
  gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

</details>

## Architecture

### Design Principles

**Single Responsibility Principle (SRP)**

Each workflow performs one specific task:
- `docker-test.yml` → Test Dockerfiles
- `docker-build.yml` → Build and push images
- `go-ci.yml` → Go continuous integration
- `helm-test.yml` → Test Helm charts
- `helm-package.yml` → Package and publish charts

**Composition Over Inheritance**

Build complex pipelines by composing simple workflows:

```yaml
jobs:
  # Run in parallel
  go-ci:
    uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@main

  # Run after go-ci
  docker-build:
    needs: go-ci
    uses: mandacode-lab/workflows/.github/workflows/docker-build.yml@main

  # Run after docker-build
  helm-package:
    needs: docker-build
    uses: mandacode-lab/workflows/.github/workflows/helm-package.yml@main
```

## Verified Tools

All tools are industry-standard, actively maintained, and widely adopted:

| Tool | Version | Purpose |
|------|---------|---------|
| [Hadolint](https://github.com/hadolint/hadolint) | v3.0.0 | Dockerfile linting |
| [Trivy](https://github.com/aquasecurity/trivy) | 0.28.0 | Security scanning |
| [golangci-lint](https://github.com/golangci/golangci-lint) | v2.2.0 | Go linting |
| [gosec](https://github.com/securego/gosec) | 2.21.4 | Go security |
| [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) | latest | Go vulnerabilities |
| [kubeconform](https://github.com/yannh/kubeconform) | 0.24.0 | K8s validation |
| [actionlint](https://github.com/rhysd/actionlint) | latest | Actions linting |
| [yamllint](https://github.com/adrienverge/yamllint) | latest | YAML linting |

## Security Features

- **Supply Chain Security**
  - SBOM (Software Bill of Materials) generation
  - Build provenance attestation
  - Artifact attestation for GHCR

- **Vulnerability Scanning**
  - Trivy for container images
  - gosec for Go code
  - govulncheck for Go dependencies
  - Automatic SARIF upload to GitHub Security

- **Best Practices**
  - Least privilege permissions
  - Secret management
  - Immutable tags for production
  - Multi-platform builds for consistency

## Migration Guide

### From Composite Workflows

**Before:**
```yaml
uses: ./.github/workflows/go_project_pull_request.yml
with:
  enable-dockerfile-test: true
  enable-helm-test: true
```

**After:**
```yaml
jobs:
  go-ci:
    uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@main

  docker-test:
    uses: mandacode-lab/workflows/.github/workflows/docker-test.yml@main

  helm-test:
    uses: mandacode-lab/workflows/.github/workflows/helm-test.yml@main
```

**Benefits:**
- Explicit and clear intent
- Easier to customize per project
- Better separation of concerns
- Simpler maintenance

## Best Practices

### Version Pinning

```yaml
# ❌ Not recommended - uses latest main
uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@main

# ✅ Recommended - pins to specific version
uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@v1.0.0
```

### Minimal Permissions

```yaml
permissions:
  contents: read        # Read repository contents
  packages: write       # Push to GHCR
  security-events: write # Upload SARIF
```

### Conditional Execution

```yaml
jobs:
  docker-test:
    if: contains(github.event.pull_request.changed_files, 'Dockerfile')
    uses: mandacode-lab/workflows/.github/workflows/docker-test.yml@main
```

### Parallel Execution

```yaml
jobs:
  # These run in parallel
  go-ci:
    uses: mandacode-lab/workflows/.github/workflows/go-ci.yml@main

  docker-test:
    uses: mandacode-lab/workflows/.github/workflows/docker-test.yml@main

  helm-test:
    uses: mandacode-lab/workflows/.github/workflows/helm-test.yml@main
```

## Troubleshooting

### Common Issues

**Workflow not found**
```
Error: Unable to resolve action `mandacode-lab/workflows/.github/workflows/go-ci.yml@main`
```
- Ensure the repository is public or you have access
- Verify the workflow file path is correct
- Check that the branch/tag exists

**Permission denied**
```
Error: Resource not accessible by integration
```
- Check the `permissions` section in your workflow
- Ensure required permissions are granted
- Verify GITHUB_TOKEN has necessary scopes

**Secret not found**
```
Error: Secret AWS_ROLE_ARN not found
```
- Add secrets in Repository Settings → Secrets and variables → Actions
- Verify secret names match exactly (case-sensitive)
- For organization secrets, check repository access

### Getting Help

1. Check the [templates README](templates/README.md) for detailed examples
2. Review workflow source code for input parameters
3. Search existing [issues](https://github.com/mandacode-lab/workflows/issues)
4. Open a new issue with workflow logs

## Contributing

Contributions are welcome! Please follow these guidelines:

### Adding a New Workflow

1. Follow the Single Responsibility Principle
2. Use verified, well-known tools only
3. Pin all versions explicitly
4. Add comprehensive documentation
5. Include usage examples in the header
6. Validate with `actionlint`

### Updating Existing Workflows

1. Maintain backward compatibility when possible
2. Update documentation and examples
3. Test thoroughly before merging
4. Consider creating a new version for breaking changes

### Workflow Template

```yaml
name: Workflow Name

# Brief description
#
# Usage example:
#   jobs:
#     workflow-name:
#       uses: mandacode-lab/workflows/.github/workflows/workflow-name.yml@main

on:
  workflow_call:
    inputs:
      # Document all inputs
    secrets:
      # Document all secrets

permissions:
  # Minimal required permissions

jobs:
  # Implementation
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by GitHub's [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- Built with best practices from the community
- Tools provided by their respective maintainers

---

**Made with ❤️ by mandacode-lab**

For more information, see the [GitHub Actions documentation](https://docs.github.com/en/actions).
