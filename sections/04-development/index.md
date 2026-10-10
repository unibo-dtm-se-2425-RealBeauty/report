---
title: Development
has_children: false
nav_order: 5
---

# Development

## DVCS

The project was developed by a single person, using Git, in the repository `artifact` of the GitHub organization. The report has its own repository, `report`.

**Branches.** All work was done on a single branch (`master` in `artifact`, `main` in `report`); no feature branches were created. With one developer and a CI pipeline that runs on every push, extra branches would only have added merge work.

**Commit messages.** All commits follow the [Conventional Commits](https://www.conventionalcommits.org/) format: `type: short description`. The types used are:

| Type | Used for | Effect on the version |
|------|----------|-----------------------|
| `feat` | a new feature (for example `feat: show the safety score out of 100`) | minor release |
| `fix` | a bug fix (for example `fix: retry when the AI provider is overloaded`) | patch release |
| `docs`, `test`, `refactor`, `build`, `ci`, `chore` | documentation, tests, code clean-up, build and pipeline changes | no release |

The format is not only a habit: the release tool (semantic-release, see [Release](../06-release/)) reads the commit messages to decide the next version number, publishes the release and writes the `chore(release)` commits by itself. At the end of the project, the repository had 39 commits and six releases (1.0.0, 1.1.0, 1.2.0, 1.2.1, 1.3.0, 1.3.1).

**Working with the commits written by the pipeline.** Because semantic-release pushes its own `chore(release)` commit to `master` after each release, the local copy is one commit behind after every release. Once, a push was rejected for this reason ("the remote contains work that you do not have locally"). It was solved with `git pull --rebase`, which places the local commits on top of the release commit and keeps the history linear, without a merge commit. Since then, `git pull` is run after every release.

**Pull requests, issues and code review.** Pull requests and issues were not used, and the code was not reviewed by a second person: changes went straight to `master`. The quality checks were automatic instead. Before each push the developer ran the formatter, the static checks and the tests locally (`poe format`, `poe static-checks`, `poe coverage`), and on every push GitHub Actions ran them again. This is a weakness for a team project, and it is listed in the [Self-evaluation](../11-selfevaluation/).

## Implementation details

**Network protocols: HTTP and HTTPS.** The browser talks to the server over HTTP, and the server talks to Open Beauty Facts and OpenRouter over HTTPS. Both external services only offer HTTP-based APIs, and the interaction is a simple request and reply. Protocols for continuous or push communication (WebSockets, MQTT, AMQP) are not needed, because the server never has to contact the browser on its own.

**Data representation: JSON, plus multipart for photos.** Requests and answers between browser and server are JSON, as are the answers of Open Beauty Facts and the exchange with the AI service. JSON is readable, supported natively by Python and by the browser, and needs no extra tool. The only exception is the photo upload, which is sent as a multipart form because it carries a file. The AI model is also instructed to answer in JSON, which the server parses.

**Database access: SQL through an ORM.** The data are simple records, so a relational database fits (see [Design](../03-design/)). SQLite is accessed through SQLAlchemy, so the code works with a Python class (`Analysis`) rather than with hand-written SQL strings. This avoids SQL injection and keeps the code short. A NoSQL database was not needed: there are no nested or variable documents and no large volumes.

**Reuse of saved results.** Before calling the AI, the server looks in the database for an earlier analysis of the same ingredient list (`find_cached_analysis` in `database.py`, called by `analyze_with_cache` in `app.py`). The comparison is done in the database query on the lower-case text, after extra spaces have been removed from the input. The answer carries a `cached` field, which the page uses to show a short neutral note ("Saved result..."), in grey rather than green so that it is not read as a judgement of the product.

**A storage bug found while adding the reuse.** Until version 1.2.1 the flagged and beneficial ingredient lists were saved with Python's `str()`, which gives text such as `[{'name': 'Parfum', ...}]` with single quotes. This is not valid JSON, so the lists could be written but not read back. The problem had gone unnoticed because nothing read these fields before. Since 1.3.0 the lists are saved with `json.dumps`. To keep the records already saved, the reading function tries JSON first and then `ast.literal_eval`, which safely reads plain Python values without running any code; a record that cannot be read in either way is treated as missing, so a new analysis is done instead.

**Authentication: none for users, an API key for the AI service.** The application has no accounts. It is a single-user tool, run locally, and the history contains no personal data. The server authenticates itself to OpenRouter with an API key, which is kept in a `.env` file outside version control. The environment variable is called `GEMINI_API_KEY` for historical reasons (the first version used Gemini); it now holds an OpenRouter key. Open Beauty Facts needs no key for reading.

**Authorization: none.** Since there are no users, there are no roles to distinguish, so RBAC or similar schemes would have no purpose. Everyone who can reach the server can use every route. This is acceptable for a local tool, but it would have to change before a public deployment (see [Future works](../12-future/)).

## Technological details

**Language and tools.** The server is written in **Python (3.10 or newer)**, because it has a mature web, database and testing ecosystem and is the language used in the course. The web page is plain **HTML, CSS and JavaScript** in a single template, without a front-end framework: the interface is one page with one form, and a framework would have been heavier than the page itself. Dependencies and packaging are managed with **Poetry**; common tasks (format, checks, tests, coverage) are defined as **poe** tasks so that they run the same way locally and in CI.

**Libraries the application depends on.**

| Library | Why |
|---------|-----|
| Flask | Small web framework: routes, templates, JSON answers, blueprints (used for the `/api/v1` prefix). |
| requests | Calls to the Open Beauty Facts API. |
| openai | Client library for OpenRouter, which offers an OpenAI-compatible API. Models can be switched by changing only a name. |
| SQLAlchemy | ORM and connection to SQLite. |
| python-dotenv | Reads the secret key from the `.env` file. |

**Libraries used only in development.**

| Library | Why |
|---------|-----|
| pytest, coverage | Running the tests and measuring coverage (see [Validation](../05-validation/)). |
| ruff | Code formatting and linting. |
| mypy | Static type checking. |
| poethepoet | The `poe` task runner. |

**External services.**

| Service | Why |
|---------|-----|
| Open Beauty Facts | Free, open database of cosmetic products: it gives name, brand and ingredients from a barcode, with no key or payment. |
| OpenRouter | One API to many AI models, including free ones. A text model is used for the analysis and a vision model for photos, with other free models as fallback. |
| GitHub (repositories, Actions, Pages) | Hosting, continuous integration, and the report website. |
| PyPI | Publishing the package `realbeauty` (see [Release](../06-release/)). |

Free AI models are not always available, so the application tries several of them in turn. This reliability limit is a direct consequence of choosing free services, and it is discussed in the [Self-evaluation](../11-selfevaluation/).
