---
type: Change
title: Update supported Python versions to 3.11-3.14
description: Dropped Python 3.10, added Python 3.14, and applied the copier-UvScaffolding v0.5.14 template updates to the CI/CD workflows.
resource: https://github.com/davidbrownell/dbrownell_Email/tree/PythonVersions
tags:
  - python-versions
  - ci-cd
  - copier-template
  - release
status: stable
generated:
  by: claude-opus-5-5
  at: 2026-10-24T10:28:08-04:00
sources:
  - id: copier-answers
    resource: /.copier-answers.yml
  - id: pyproject
    resource: /pyproject.toml
  - id: cicd
    resource: /.github/workflows/CICD.yml
  - id: cicd-impl
    resource: /.github/workflows/CICD_impl.yml
  - id: release-action
    resource: /.github/actions/release_impl/action.yml
  - id: template
    resource: https://github.com/gt-csse/copier-UvScaffolding
---

# Summary

The package now supports Python 3.11, 3.12, 3.13, and 3.14. Python 3.10 support was dropped. The project was regenerated from copier-UvScaffolding `v0.5.14` (previously `v0.5.11`)[^copier-answers][^template], which also brought a set of CI/CD reliability and security improvements.

# Motivation

- Python 3.10 reaches end of life in October 2026, and Python 3.14 is the current stable release. The supported range now tracks the actively maintained CPython versions.
- Updating to the latest scaffolding template keeps the repository consistent with other projects generated from it and picks up fixes to the release pipeline.

# Changes

## Python versions

- `requires-python` raised from `>= 3.10` to `>= 3.11`. The trove classifiers now list 3.11-3.14[^pyproject].
- The CI test matrix changed from `3.10-3.13` to `3.11-3.14`. The package build Python changed from 3.10 to 3.11[^cicd].
- `uv.lock` was regenerated for `>=3.11`, which removed the cp310 wheels and the `typing-extensions` marker dependency for `cryptography`.

## Build configuration

- `uv_build` was raised to `>=0.13.5,<0.14.0`.
- The unused `[tool.hatch.version]` section was removed.
- pytest now also writes an HTML coverage report.
- Author name metadata was updated, and the ruff ignore-list comments were realigned.
- The `dbrownell_CommitEmojis` dev dependency was upgraded from 0.1.4 to 0.5.5 in `uv.lock`, which adds the `DisplayJson` command.

## CI/CD workflow

- A concurrency group was added[^cicd]. Superseded pull-request runs are cancelled. Each push gets a unique group, so a release run is never cancelled.
- Every job now has a `timeout-minutes` limit[^cicd-impl].
- Some names were changed: `python_package_version` became `build_python_version`, `package_coverage` became `postprocess_coverage`, and `package_name` became `wheel_filename`.
- The redundant `uv sync --frozen` steps were removed.
- The build fails unless exactly one wheel is produced.
- Values are passed through environment variables instead of being interpolated `${{ }}` into shell scripts. This prevents expression injection.

## Release action

- The release values (wheel filename, package version, coverage total) are now explicit action inputs instead of implicit job environment variables[^release-action].
- A new `retry.sh` retries tag pushes and GitHub release creation up to 3 times.
- If the GitHub release already exists, assets are uploaded to it, so a rerun can recover from a partial failure.
- The minisign private key is written to a `mktemp` file under `RUNNER_TEMP` and removed by an `EXIT` trap, instead of a fixed `/tmp/minisign.key`. All artifacts are now signed in a single Python invocation.
- The coverage badge is skipped when the coverage total is empty.

## Documentation

- `README.md` names the signature extension correctly as `.minisig` (previously `.minisign`).
- `DEVELOPMENT.md` has a new Semantic Versioning section and an updated signing command.
- `.github/release_sources.yaml` and `src/AutoGitSemVer.yaml` each have a comment requiring them to stay in sync.

# Impact

- Users on Python 3.10 can no longer install new releases. Earlier releases remain available to them.
- There is no change to the runtime behavior of the package.

[^copier-answers]: `.copier-answers.yml`
[^template]: copier-UvScaffolding template
[^pyproject]: `pyproject.toml`
[^cicd]: `.github/workflows/CICD.yml`
[^cicd-impl]: `.github/workflows/CICD_impl.yml`
[^release-action]: `.github/actions/release_impl/action.yml`
