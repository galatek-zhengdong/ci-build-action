# ci-build-action

Reusable GitHub Action for running Makefile-based Docker image packaging jobs.

This action prepares the runtime environment for a packaging Makefile, then runs a selected Make target from a selected project directory. It can configure SSH access for private source repositories and log in to a private Docker registry before the Makefile runs.

## Capabilities

- Validate that the requested project directory exists and contains a `Makefile`.
- Configure an SSH key for Makefile-driven `git clone` or `git pull` commands.
- Populate `~/.ssh/known_hosts` from explicit entries or `ssh-keyscan`.
- Log in to a Docker registry with secret credentials.
- Optionally run `make clone` before the build target.
- Run a selected Make target such as `develop`, `test`, `demo`, `release`, or `kunpeng`.
- Expose the executed project directory, target, and Make command as outputs.

## Quick Start

```yaml
name: Build image

on:
  workflow_dispatch:
    inputs:
      project:
        description: Project directory under build-projects.
        required: true
        default: example-service
        type: choice
        options:
          - example-service
          - example-web
          - example-worker
      target:
        description: Make target.
        required: true
        default: develop
        type: choice
        options:
          - develop
          - test
          - demo
          - release
          - kunpeng

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout packaging configuration
        uses: actions/checkout@v4

      - name: Run Makefile packaging target
        uses: galatek-zhengdong/ci-build-action@v1
        with:
          project-dir: build-projects/${{ inputs.project }}
          target: ${{ inputs.target }}
          docker-registry: registry.example.com
          docker-username: ${{ secrets.REGISTRY_USERNAME }}
          docker-password: ${{ secrets.REGISTRY_PASSWORD }}
          ssh-private-key: ${{ secrets.SOURCE_REPO_SSH_KEY }}
```

See [examples/manual-build.yml](examples/manual-build.yml) for a longer workflow example.

## Execution

With default inputs, the action runs:

```shell
make -C <project-dir> clone
make -C <project-dir> <target>
```

Set `clone: false` when the runner already has the required build source available and the selected target should handle source updates itself.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `project-dir` | yes | | Directory containing the packaging `Makefile` |
| `target` | no | `develop` | Make target to execute |
| `make-args` | no | | Extra arguments appended to the Make command |
| `clone` | no | `true` | Run `make clone` before the selected target |
| `docker-registry` | no | | Docker registry host |
| `docker-username` | no | | Docker registry username |
| `docker-password` | no | | Docker registry password or token |
| `ssh-private-key` | no | | SSH private key for Makefile-driven Git commands |
| `ssh-known-hosts` | no | | Explicit `known_hosts` entries |
| `ssh-keyscan-hosts` | no | `github.com` | Hosts scanned when `ssh-known-hosts` is empty |

## Outputs

| Name | Description |
| --- | --- |
| `project-dir` | Makefile project directory |
| `target` | Executed Make target |
| `make-command` | Build target command executed by the action |

## Secrets

Typical callers provide:

- `SOURCE_REPO_SSH_KEY`: SSH key with read access to private source repositories.
- `REGISTRY_USERNAME`: Docker registry username.
- `REGISTRY_PASSWORD`: Docker registry password or token.

## Release

Publish a stable version tag:

```shell
git tag v1
git push origin v1
```

Callers should reference a stable tag:

```yaml
uses: galatek-zhengdong/ci-build-action@v1
```
