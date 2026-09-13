# .github

Shared GitHub Actions workflows and organization-wide configuration for
[aistandardsio](https://github.com/aistandardsio).

This repo is public, and holds only generic, mechanical workflow
definitions — nothing sensitive. Architecture rationale, the private
cross-repo dependency graph, and operational details live in the private
[`internal-aistandardsio-system`](https://github.com/aistandardsio/internal-aistandardsio-system)
repo instead (see its `infrastructure/ci.md`).

## Reusable Workflows

### go-ci.yaml

Cross-platform Go testing.

```yaml
jobs:
  ci:
    uses: aistandardsio/.github/.github/workflows/go-ci.yaml@main
    secrets:
      PRIVATE_REPO_TOKEN: ${{ secrets.PRIVATE_REPO_TOKEN }}
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-versions` | `["1.26.x"]` | JSON array of Go versions |
| `platforms` | `["ubuntu-latest", "macos-latest", "windows-latest"]` | JSON array of platforms |
| `test-flags` | `-v -covermode=count` | Additional `go test` flags |

### go-lint.yaml

Go linting with golangci-lint, using the canonical
[`plexusone/.github`](https://github.com/plexusone/.github) `.golangci.yaml`
(fetched at lint time, not vendored per-repo).

```yaml
jobs:
  lint:
    uses: aistandardsio/.github/.github/workflows/go-lint.yaml@main
    secrets:
      PRIVATE_REPO_TOKEN: ${{ secrets.PRIVATE_REPO_TOKEN }}
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-version` | `1.26.x` | Go version |
| `golangci-lint-version` | `latest` | golangci-lint version |
| `timeout` | `3m` | Lint timeout |
| `args` | `--verbose` | Additional golangci-lint arguments |

### go-sast-codeql.yaml

CodeQL static analysis.

```yaml
jobs:
  sast:
    uses: aistandardsio/.github/.github/workflows/go-sast-codeql.yaml@main
    secrets:
      PRIVATE_REPO_TOKEN: ${{ secrets.PRIVATE_REPO_TOKEN }}
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-version` | `1.26.x` | Go version |
| `queries` | `security-extended,security-and-quality` | CodeQL query suites |

## `PRIVATE_REPO_TOKEN`

All three workflows accept an **optional** `PRIVATE_REPO_TOKEN` secret: a PAT
with read access to this org's private repos, used to resolve private
`aistandardsio` Go module dependencies (e.g. `aisecurity-schemas` importing
`aisecurity-core`) that the default `GITHUB_TOKEN` can't reach. Repos with no
private `aistandardsio` dependencies can omit it. Pass it via an
organization-level Actions secret with `secrets: inherit`, or explicitly as
shown above. See `internal-aistandardsio-system/infrastructure/ci.md` for
why this is needed and how the token should be scoped.

## Versioning

Use `@main` for latest or pin to a specific tag once tags exist:

```yaml
uses: aistandardsio/.github/.github/workflows/go-ci.yaml@v0.1.0
```
