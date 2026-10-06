---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

RealBeauty is a web application meant to be run by the person who uses it, on their own computer. It is not hosted on a public server. A user therefore installs and starts it locally; the browser and the server run on the same machine.

## User installation

**What has to be installed.** The following must be present on the machine:

| Software | Version | Why |
|----------|---------|-----|
| Python | 3.10 or newer | The application is written in Python. |
| Poetry | any recent version | Installs the dependencies in an isolated environment and starts the application. |
| Git | any | Downloads the source code. |
| A web browser | any recent one | Used as the interface. |

Poetry can be installed with `pipx install poetry` (or `pip install poetry`). The libraries the application needs (Flask, requests, openai, SQLAlchemy, python-dotenv) are installed automatically by Poetry in the next step.

An **OpenRouter API key** is also needed, because the analysis is done by AI models reached through OpenRouter. The key is obtained by creating a free account on [openrouter.ai](https://openrouter.ai/) and creating a key in the account settings. No other account or payment is required, since the application uses free models.

**How to install.**

```bash
git clone https://github.com/unibo-dtm-se-2425-RealBeauty/artifact.git
cd artifact
poetry install
```

The first command downloads the code, the second enters the folder, and the third creates the environment and installs the libraries.

The package is also published on PyPI (`realbeauty`), but the HTML page lives outside the Python package. Running from the downloaded source code, as above, is therefore the supported way to use the web application.

**How to configure.** The only configuration is the key. A file named `.env` must be created in the project folder (the file is excluded from version control, so the key is never published) with this content:

```
GEMINI_API_KEY=your-openrouter-api-key
```

The variable has this name for historical reasons (the first version used a Gemini key); it holds the OpenRouter key.

**How to start.**

```bash
poetry run flask --app artifact.app run
```

and open <http://127.0.0.1:5000> in the browser. On macOS, port 5000 is sometimes taken by the AirPlay receiver; in that case the application is started on another port with `--port 5001` and opened at <http://127.0.0.1:5001>.

The database file (`realbeauty.db`) is created automatically in the project folder on the first start. To remove all saved analyses it is enough to stop the application and delete this file.

## Server-side installation

**The software does not need a dedicated server.** The "server" of the architecture (see [Design](../03-design/)) is the Flask process that runs on the user's own machine, started with the command above, so the installation steps are the same as in the previous section.

**No additional software is needed.** In particular:

- there is **no database server**: SQLite is built into Python and stores the data in one file;
- there is **no message broker, cache or web server**, because the design does not use them;
- the two external services (Open Beauty Facts and OpenRouter) are used over the Internet and need no installation, only an Internet connection.

**Hosting on a public server.** This was not done, and the setup above is not suitable for it. The command used to start the application runs Flask's built-in development server, which is meant for a single user and is not hardened against public traffic. Moreover, the application has no user accounts (see [Development](../04-development/)), so everyone who reached the server could use the key of the owner and fill the history. Before a public deployment, a production web server (for example Gunicorn behind a reverse proxy with HTTPS), a container image, user authentication and a request limit would have to be added. These steps are listed in [Future works](../12-future/).

**The report website.** Separately from the software, this report is published as a website by GitHub Pages, which builds the Markdown files of the `report` repository automatically at every push to `main`. No installation is needed to read it.
