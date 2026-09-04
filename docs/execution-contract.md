# Execution Contract

`ci-build-action` is a composite GitHub Action that runs Makefile-based Docker image packaging jobs. It does not own project-specific build rules. The caller provides a checkout that contains one or more packaging Makefiles.

## Required Caller State

Before this action runs, the caller must check out a repository that contains the selected `project-dir`.

The selected directory must contain:

- `Makefile`
- any files included by that Makefile
- any project-specific packaging configuration required by that Makefile

## Runtime Preparation

The action can prepare two shared runtime concerns before invoking Make:

- SSH access for Makefile-driven Git commands.
- Docker registry authentication for Makefile-driven image pushes.

Both are optional. If the corresponding inputs are empty, the action skips that setup step.

The built-in Docker login step supports one registry. Workflows that need multiple registries should run additional `docker login` steps before invoking this action. Those credentials remain available in the runner's Docker config for the Makefile commands executed by this action.

## Command Order

When `clone` is `true`, the action runs:

```shell
make -C <project-dir> clone
```

Then it runs the selected target:

```shell
make -C <project-dir> <target> <make-args>
```

The action fails immediately if either Make command fails.

## Target Contract

The action treats `target` as an opaque Make target name. It does not interpret environment names, branches, Docker image names, or tag rules. Those rules belong to the Makefile selected by `project-dir`.

## Security Contract

Secret values must be provided through GitHub Actions secrets. The action accepts secrets as inputs only so it can pass them to `ssh-agent` or `docker login`; it does not write them to action outputs.
