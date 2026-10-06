---
title: Release
has_children: false
nav_order: 7
---

# Release

## Artefacts and release process

**Artefacts.** Every release produces the following artefacts from the codebase:

| Artefact | Description | Where it is published |
|----------|-------------|-----------------------|
| Python package `realbeauty` | The application code (the `artifact` package), built as a source archive and a wheel (`dist/*`). | [PyPI](https://pypi.org/project/realbeauty/) |
| GitHub release | A tagged release with automatically generated release notes and the files of `dist/` attached. | GitHub, repository `artifact` |
| `CHANGELOG.md` | The history of all releases, grouped into features, bug fixes and documentation. | In the repository |

PyPI is the standard repository for Python packages and is free. GitHub is where the code already lives, so the release notes and the tags stay next to the source. No Docker image was produced (see [Deployment](../07-deployment/)).

**How a release is made: automatically.** No release step is done by hand. The release is the last step of the CI/CD pipeline (see [CI/CD](../08-cicd/)) and works as follows:

1. A commit is pushed to the `master` branch.
2. The pipeline runs the static checks, the format check and the tests on all operating systems and Python versions.
3. Only if all of them pass, the *deploy* job starts. It installs Node.js and runs `npx semantic-release`.
4. semantic-release reads the commit messages since the last release and decides whether a new version is needed and which one (see *Versioning* below). If the commits contain only documentation, tests or similar changes, nothing is released.
5. If a release is needed, semantic-release sets the new version in `pyproject.toml` (`poetry version`), builds and uploads the package (`poetry publish --build`), creates the Git tag and the GitHub release with the notes, and finally commits the updated `CHANGELOG.md` and `pyproject.toml` back to the repository with a message such as `chore(release): 1.2.1 [skip ci]`. The `[skip ci]` part prevents the pipeline from starting again for that commit.

The behaviour is defined in `release.config.mjs` and `.github/workflows/deploy.yml`. On a branch other than `master` or `main`, the release runs in *dry-run* mode: it shows what would be released without publishing anything.

**One-time configuration.** To make the pipeline work, two secrets were added to the repository settings (*Settings, Secrets and variables, Actions*):

| Secret | Content |
|--------|---------|
| `PYPI_TOKEN` | An API token created in the PyPI account, allowed to upload the package. |
| `RELEASE_TOKEN` | A GitHub access token that may create releases and push the release commit. |

The tokens are never stored in the code. No command has to be run to release: pushing a `feat` or `fix` commit is enough.

## Choice of the license

**License: Apache License 2.0**, for both the code and the published package. The `LICENSE` file is in the repository root and the package metadata (`pyproject.toml`) declares the same license.

Reasons:

- It is a *permissive* license: anyone may use, modify and redistribute the code, also in other projects, as long as the license text and the notices are kept.
- It contains an explicit patent grant, which the shorter MIT license does not.
- It is the license of the project template used for the course, and nothing in the project (all dependencies are under permissive licenses as well) required a different choice.

There is only one artefact, so the same license applies to everything that is published. The data used at run time belong to Open Beauty Facts and keep their own license (Open Database License), and the AI models belong to their providers. Neither is distributed with the package.

## Choice of the versioning schema

**Schema: Semantic Versioning (SemVer), `MAJOR.MINOR.PATCH`.** It tells users at a glance what kind of change a new version contains:

| Part | Increased when | Triggered by the commit type |
|------|----------------|------------------------------|
| MAJOR (`2.0.0`) | A change breaks compatibility with the previous version. | `BREAKING CHANGE` in the message |
| MINOR (`1.3.0`) | A new feature is added, compatibly. | `feat:` |
| PATCH (`1.2.2`) | A bug is fixed, compatibly. | `fix:` |

Commits of type `docs`, `test`, `refactor`, `build`, `ci` and `chore` do not create a release. Date-based versioning was not chosen because the version should describe the *kind* of change, not only when it happened.

**Number of artefacts.** All artefacts (package, tag, GitHub release) come from the same release step and carry the same number, so there is nothing to align.

**Relation to the API version.** The package version (for example `1.2.1`) and the version of the HTTP API (`/api/v1`) are different things. The API version changes only when the API changes in an incompatible way; the package version changes with every release.

**How to create a new version.** There is no manual procedure:

1. Make the change and commit it with a Conventional Commits message (`feat: ...` for a new feature, `fix: ...` for a bug fix; add a line `BREAKING CHANGE: ...` for an incompatible change).
2. Push to `master`.
3. When the pipeline is green, the new version appears on PyPI and in the GitHub releases, together with the tag (for example `1.2.1`, without a `v` prefix).

No release branch is needed, because only one version is maintained: the latest one.

**Releases made so far.**

| Version | Main content |
|---------|--------------|
| 1.0.0 | First complete version: analysis from a barcode, from typed ingredients and from a label photo, with saved history. |
| 1.1.0 | The HTTP API versioned under `/api/v1`; logging of AI failures; longer timeout for AI requests. |
| 1.2.0 | Score shown out of 100; retries when the AI provider is overloaded; ingredients read from language-specific fields. |
| 1.2.1 | Fallback to alternative free models; detection of the real image type of uploaded photos. |
