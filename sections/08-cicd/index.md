---
title: CI/CD
has_children: false
nav_order: 9
---

# CI/CD

## Conceptual description

Everything that can be checked or published by a program is automated with **GitHub Actions**. The goal is that the quality of the code does not depend on the developer remembering to run something: every push is verified in the same way, on clean machines, and a release happens only if the verification succeeds.

The pipeline is defined in two files of the `artifact` repository, `.github/workflows/check.yml` (named *CI/CD*) and `.github/workflows/deploy.yml`, and has three stages that run one after the other. A stage starts only if the previous one succeeded.

| Stage | Job | Runs on | What is automated | Why |
|-------|-----|---------|-------------------|-----|
| 1. Preliminary checks | `check` | Ubuntu, one Python version | Syntax check, static analysis, format check, tests with coverage | Fast feedback: the cheapest and most common problems are found first. |
| 2. Tests everywhere | `test` | 3 operating systems × 4 Python versions (12 runs) | The test suite | The application should work on the systems a user may have (requirement NFR7). |
| 3. Release | `deploy` | Ubuntu | Choice of the version, package upload to PyPI, tag, GitHub release, changelog | A release is published without manual steps, and only from code that passed everything above. |

If a stage fails, the next one does not start, and nothing is published.

## How it works

**When the pipeline runs.** It starts on every push, except pushes to branches created by the dependency bots (`dependabot/**`, `renovate/**`) and pushes that change only files with no effect on the code (`README.md`, `CHANGELOG.md`, `LICENSE`, `.gitignore`, `renovate.json`, `.mergify.yml`). It also starts on pull requests and can be started manually from the Actions page (`workflow_dispatch`). The commit that the release tool creates at the end has `[skip ci]` in its message, so it does not start the pipeline again.

**Stage 1: preliminary checks.** On a clean Ubuntu machine the job checks out the full history, installs Poetry and the project dependencies (`poetry install`), and then runs, in this order:

| Step | Command | Purpose |
|------|---------|---------|
| Syntax | `poe compile` | All Python files can be compiled. |
| Static checks | `poe static-checks` | `ruff` (style and common errors) and `mypy` (types). |
| Format | `poe format-check` | The code is formatted as `ruff format` would do. |
| Tests with coverage | `poe coverage`, `poe coverage-report`, `poe coverage-html` | Runs the tests while measuring which lines run, and prints the report. |

The HTML coverage report is saved as a downloadable *artifact* of the run (`coverage-report-<commit>`). The same `poe` tasks are used on the developer's machine, so a check that passes locally behaves the same in the pipeline.

**Stage 2: test matrix.** The test suite is run on Ubuntu, Windows and macOS, each with Python 3.10, 3.11, 3.12 and 3.13, i.e. 12 combinations. `fail-fast` is disabled, so a failure in one combination does not hide the result of the others, and every job has a 45-minute time limit. The tests need no network and no API key, because the outside services are replaced by test doubles (see [Validation](../05-validation/)).

**Stage 3: release.** The `deploy` job is defined as a reusable workflow in `deploy.yml` and is called by the main workflow after all test jobs have passed. It installs Node.js (the version is read from `package.json`) and runs `npx semantic-release`, which decides the new version from the commit messages and publishes it, as described in [Release](../06-release/). Details:

- A `concurrency` group allows only one release job at a time, so two quick pushes cannot publish at the same moment.
- The release is a **dry run** (nothing is published) when the branch is not `master` or `main`, and for the very first commit of a repository.
- The upload to PyPI is done with `poetry publish --build`.

**Secrets and variables.** Secrets are stored in the repository settings and are passed to the release step as environment variables; they never appear in the code or in the logs.

| Name | Where it is used | Purpose |
|------|------------------|---------|
| `PYPI_TOKEN` | Release step | Uploads the package to PyPI. |
| `RELEASE_TOKEN` | Release step, passed as `GITHUB_TOKEN` | Creates the tag and the GitHub release and pushes the release commit. |
| `RELEASE_DRY_RUN` | Release step (computed) | `true` on other branches and on the first commit. |

The application's own key for the AI service is **not** a secret of the pipeline: the tests do not call the service, so the pipeline does not need it.

**Other automation.** The file `renovate.json` configures a bot that proposes updates of the dependencies. GitHub's dependency alerts are also active (see the open points in [Self-evaluation](../11-selfevaluation/)). The report repository has no pipeline of its own: GitHub Pages builds the website from the Markdown files at every push to `main`.

**Status.** A badge in the README shows whether the last run of the pipeline succeeded. At the time of writing, all stages are green and four releases (1.0.0 to 1.2.1) were produced by it.
