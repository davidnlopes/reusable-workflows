# reusable-workflows

Shared GitHub Actions workflows that other repositories call to build, test, scan, and deploy their code. A repository adds a few lines of YAML instead of maintaining its own pipeline, and every caller gets the same checks, pinned actions, and least-privilege permissions.

Each workflow uses the `workflow_call` trigger. Workflows that run on pull requests report a commit status, such as `Golang CI`, that you can mark as required in branch protection.

## Workflows

| Workflow | What it does | Secrets |
| --- | --- | --- |
| `golang.yaml` | Runs your build and test commands, then uploads coverage to Codecov | `CODECOV_TOKEN` (optional) |
| `python.yaml` | Runs your setup and test commands, then uploads coverage to Codecov | `CODECOV_TOKEN` |
| `react.yaml` | Installs dependencies with npm, pnpm, or Yarn, then lints, tests, builds, and uploads coverage | `CODECOV_TOKEN` |
| `terraform.yaml` | Formats, validates, and runs `plan`, `apply`, or `destroy` with Terraform or OpenTofu, and comments the results on the PR | None, uses AWS OIDC |
| `docker.yaml` | Builds the targets of your `docker-bake.hcl` with Docker Buildx Bake, and pushes them to a registry when asked | `REGISTRY_PASSWORD` (optional), or AWS OIDC for Amazon ECR |
| `trivy-scan.yaml` | Scans the repository with Trivy and comments the findings on the PR | None |
| `pre-commit-hooks.yaml` | Runs the repository's pre-commit hooks | None |
| `semantic-pr.yaml` | Checks that the PR title follows conventional commits and that the PR has exactly one `major`, `minor`, or `patch` label | None |
| `labeler.yaml` | Labels the PR from the changed files, using the shared label rules | None |

Every input, with its default and description, is declared in the `on.workflow_call` block at the top of each file in [`.github/workflows/`](.github/workflows/).

## Usage

Call a workflow from a job in your repository:

```yaml
# .github/workflows/ci.yaml
name: CI
on: [pull_request]

jobs:
  golang:
    uses: davidnlopes/reusable-workflows/.github/workflows/golang.yaml@v1
    with:
      buildCommand: go build ./...
      testCommand: go test -coverprofile=coverage.out ./...
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Every release is tagged three ways, so you choose how much change you accept: `@v1` follows the latest release of that major version, `@v1.2` follows patch releases only, and `@v1.2.3` never changes.

Most workflows also share a few inputs:

- `ignoreErrors` reports success even when the checks fail. Use it to roll out a new check without blocking PRs.
- A directory input, such as `golangProjectDir`, `terraformProjectDir`, or `projectDir`, runs the workflow in a subdirectory of a monorepo.
- `runsOn` picks the runner, and defaults to `ubuntu-latest`.

`terraform.yaml` and `docker.yaml` sign in to AWS with GitHub OIDC, so they need no stored AWS keys. Create an IAM role that trusts GitHub's OIDC provider for your repository, and pass its ARN as `awsRoleArn`.

## Documentation

| Topic | Document | Read it when |
| --- | --- | --- |
| Architecture | [`docs/architecture.md`](docs/architecture.md) | You want to know how the workflows, the shared action, and external services fit together |
| Domain | [`docs/domain.md`](docs/domain.md) | You meet an unfamiliar term or need the rules every workflow follows |
| Patterns | [`docs/patterns.md`](docs/patterns.md) | You add or change a workflow |
| Development | [`docs/development.md`](docs/development.md) | You test a change, open a PR, or want to know how releases work |

## Development

There is nothing to build or install locally. To test a change, push a branch and call the workflow from a test repository by branch name:

```yaml
uses: davidnlopes/reusable-workflows/.github/workflows/golang.yaml@your-branch
```

Open the PR with a conventional commit title and exactly one `major`, `minor`, or `patch` label. Merging to `main` publishes a release. [`docs/development.md`](docs/development.md) has the full process.
