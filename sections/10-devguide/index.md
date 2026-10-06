---
title: Developer guide
has_children: false
nav_order: 11
---

# Developer Guide

This guide is for a person who wants to continue the development of RealBeauty or to fix something in it. It describes how the project is organised, how to set up the development environment, and which rules to follow.

## Organisation

The project was developed by a single person, who is also the contact for questions: Serra Özkasap, <serra.ozkasap@studio.unibo.it>. Both repositories are in the GitHub organization `unibo-dtm-se-2425-RealBeauty`:

| Repository | Content | Main branch |
|------------|---------|-------------|
| `artifact` | The application, its tests and the CI/CD pipeline. | `master` |
| `report` | This report (a Jekyll website published with GitHub Pages). | `main` |

Issues have not been used so far in the project. A problem found by a future contributor can be reported as an issue of the `artifact` repository (tab *Issues*), giving the steps to reproduce it, what was expected and what happened. Known limitations are listed in [Self-evaluation](../11-selfevaluation/) and ideas for improvement in [Future works](../12-future/); they are a good starting point.

## Development environment

**Requirements:** Python 3.10 or newer, Git, [Poetry](https://python-poetry.org/), and an OpenRouter key for running the application (the tests do not need it).

```bash
git clone https://github.com/unibo-dtm-se-2425-RealBeauty/artifact.git
cd artifact
poetry install
```

`poetry install` creates an isolated environment and installs both the application and the development tools. The key is put in a `.env` file as described in [Deployment](../07-deployment/). To run the application:

```bash
poetry run flask --app artifact.app run
```

The server has to be restarted after each change of the code, because the automatic reload is not enabled (the option `--debug` enables it).

**Project structure.**

| Path | Content |
|------|---------|
| `artifact/app.py` | Flask application and routes (blueprint `api_v1`, prefix `/api/v1`). |
| `artifact/analyzer.py` | Calls to the AI models: analysis, reading of photos, retries and fallback. The lists `TEXT_MODELS` and `VISION_MODELS` and the timeouts are at the top of the file. |
| `artifact/beauty_api.py` | Client of Open Beauty Facts. |
| `artifact/database.py` | SQLite database and the `Analysis` table. |
| `templates/index.html` | The whole web interface (HTML, CSS and JavaScript). |
| `tests/` | One test file per module. |
| `pyproject.toml` | Dependencies and the `poe` tasks. |
| `.github/workflows/` | CI/CD pipeline. |
| `release.config.mjs` | Configuration of the automatic release. |

The role of each module is explained in [Design](../03-design/).

## Commands

All common operations are defined as `poe` tasks, so they are the same on every machine and in the CI pipeline.

| Command | What it does |
|---------|--------------|
| `poetry run poe test` | Runs the tests. |
| `poetry run poe coverage` | Runs the tests while measuring coverage. |
| `poetry run poe coverage-report` | Prints the coverage report (lines not covered included). |
| `poetry run poe format` | Formats the code with `ruff format`. |
| `poetry run poe format-check` | Checks the formatting without changing files. |
| `poetry run poe static-checks` | Runs `ruff` (style and errors) and `mypy` (types). |
| `poetry run poe compile` | Checks that all files can be compiled. |

**Before every push**, the following must pass (they are the same checks as in the pipeline):

```bash
poetry run poe format
poetry run poe static-checks
poetry run poe coverage
```

## Conventions

**Code style.** The formatting is enforced by `ruff format` and the style rules by `ruff check`, so no manual decision is needed: running `poe format` is enough. Type hints are checked by `mypy`. Modules and functions are named in `snake_case`; constants in `UPPER_CASE`. Error messages shown to users are in English.

**Where to put new code.**

- A new HTTP route goes in `app.py`, on the `api_v1` blueprint. A route that changes the behaviour of an existing one in an incompatible way requires a new version (`/api/v2`).
- Everything that talks to an outside service goes in its own module (like `beauty_api.py` and `analyzer.py`), not in `app.py`. The API layer only coordinates.
- A change to the stored data goes in `database.py`. Note that the database file is not migrated: when the table changes, an existing `realbeauty.db` must be deleted or adapted.
- To use other AI models, only the lists in `analyzer.py` need to be changed. The free models available on OpenRouter change often.

**Tests.** Every new behaviour needs a test in `tests/`. Outside services are never called in tests: the functions that call them are replaced with `unittest.mock.patch` (see the existing tests for examples). A test should be named after the behaviour it checks, for example `test_analyze_barcode_not_found_returns_404`. The minimum coverage is 70%, and the current value is higher (see [Validation](../05-validation/)).

**Commit messages.** [Conventional Commits](https://www.conventionalcommits.org/): `type: short description in the imperative`, for example `fix: retry when the AI provider is overloaded`. The type matters, because it decides the next version:

| Type | Meaning | Release |
|------|---------|---------|
| `feat` | new feature | minor |
| `fix` | bug fix | patch |
| `docs`, `test`, `refactor`, `build`, `ci`, `chore` | no change of behaviour for the user | none |

An incompatible change is marked with a line `BREAKING CHANGE: ...` in the body of the message, and produces a major release.

## Workflow

The project itself was developed directly on `master`, as described in [Development](../04-development/). A new contributor is advised to use a short-lived branch and a pull request instead; the pipeline also runs on pull requests, so the checks are done before the code reaches `master`:

1. Create a branch: `git checkout -b feat/short-name`.
2. Make the change, with its tests, and run the three commands listed above.
3. Commit with a Conventional Commits message and push the branch.
4. Open a pull request towards `master` and wait for the pipeline to be green.
5. After the merge, the pipeline runs on `master`. If the commits justify it, a new version is published automatically (see [Release](../06-release/)): no manual tagging or uploading is needed.

Secrets (`PYPI_TOKEN`, `RELEASE_TOKEN`) are configured in the repository settings and must never be written in the code. The `.env` file is excluded from Git; if a key is ever committed by mistake, it has to be revoked in OpenRouter and replaced, because removing the commit is not enough.

**Working on the report.** The report is edited in the `report` repository. The Markdown files are in `sections/`; the front matter at the top of each file (title, `nav_order`) must be kept. Images are in `pictures/` and are referenced with `../../pictures/<name>.png`. GitHub Pages rebuilds the website after each push to `main`, which takes a minute or two.

## Tools

The project was developed with Visual Studio Code and the macOS Terminal, but no specific editor is required. The editor should use the project's Poetry environment (in VS Code: *Python: Select Interpreter*, then the one inside `.venv`), so that `ruff` and `mypy` find the installed libraries. The test of a single file or function can be run with `poetry run pytest tests/test_app.py -k not_found`, and the application can be tried without the browser with `curl` (an example is in the README).
