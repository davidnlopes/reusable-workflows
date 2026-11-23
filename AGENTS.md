# reusable-workflows

Reusable GitHub Actions workflows that other repositories call, so that many repositories share one CI/CD pipeline instead of each maintaining its own. Nothing here builds or runs locally: the only artefacts are the workflows in `.github/workflows/`, one composite action in `.github/actions/`, and the release configuration. Consumers call them by a version tag, so every change is a change to a public interface.

## Two kinds of workflow

Consumer-facing workflows use the `workflow_call` trigger. Their `on.workflow_call` block is the public API: input names, types, and defaults are contract, and renaming or removing one is a breaking change for every caller. The workflows that run on this repository's own events (`release.yaml`, `actionlint.yaml`, `docker-test.yaml`) are internal and are not part of that API.

Consumer-facing workflows that run a caller's checks report them as a commit status named by `CHECK_NAME`, in three phases: pending first, then the checks, then success or failure. This gives the PR immediate feedback and lets callers require the check in branch protection.

Tools such as Go, Terraform, Trivy, and pre-commit are installed by the composite action `install-common-tools`. Workflows reach it by checking out this repository into `actions/`. Extend that action when a workflow needs another tool, instead of installing it inline.

## Releases

Merging to `main` runs semantic-release, which reads conventional commit types to pick the version and moves the `vX` and `vX.Y` tags. The PR title becomes the commit, so the title decides the release. See [docs/development.md](docs/development.md) before you open a PR or test a change from a consumer repository.

## What must stay true

- Every third-party action is pinned by commit SHA, with the version in a trailing comment (`uses: owner/action@<sha> # v1.2.3`). Callers run these workflows with their own secrets and tokens, so a movable tag would let an upstream change run in their repositories. Actions referenced by local path are exempt.
- Permissions are declared at job level, never for the whole workflow, and each line carries a comment saying why it is needed. Callers inherit whatever a job requests, so each job asks for the least it needs.
- A workflow that runs a caller's checks and sets a commit status accepts `ignoreErrors`, and reports success when it is set. `semantic-pr.yaml` and `labeler.yaml` do not have the input.
- Workflows that run a project's commands do so inside a caller-chosen directory (`defaults.run.working-directory` set from a directory input), so that monorepos can use them.

## Where details live

- [docs/architecture.md](docs/architecture.md) for how workflows, the composite action, and external services connect.
- [docs/patterns.md](docs/patterns.md) before you add or restructure a workflow; it shows the status-check, permissions, and input conventions in code.
- [docs/domain.md](docs/domain.md) for the project's terms.
