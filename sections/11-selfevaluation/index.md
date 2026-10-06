---
title: Self-evaluation
has_children: false
nav_order: 12
---

# Self-evaluation

I carried out this project alone, so this is the only self-evaluation section.

## My role and how the work went

I was responsible for everything: the idea, the requirements, the architecture, the code, the tests, the CI/CD pipeline, the releases and this report.

**The idea.** It started from something I do often: checking what is inside a personal care product before buying or using it. I wanted to see how this could be done in an easier way, with an application that gives an answer in a few seconds.

**How the inputs were decided.** My first tests used a barcode and the Open Beauty Facts database. The database is large, but I soon saw that it does not contain every product, or that a product has no ingredient list. For this reason I added the possibility to type the ingredients. Later I noticed that typing a long list is a waste of time. Since an AI model was already needed for the analysis, I thought that it could also read the list from a photo of the label, and I added the photo input. Each input was added because a real problem appeared in my own tests.

**Choice of the architecture.** In the Software Engineering course I had seen several architectural styles. I went through them one by one and asked, for each, what it would give to this application. Event-based and service-oriented styles would only add infrastructure, and an object-oriented design had too little domain to work on. A layered client-server structure was the one that matched the way the application works: one request, a few steps one after the other, one answer. The reasoning for each style is in the [Design](../03-design/) section.

**The AI analysis and the prompt.** I looked at several AI services before deciding how to do the analysis. Using an Anthropic model was not possible within the project because of its cost and usage quotas, so I had to move to other services. The first version used a Gemini model through OpenRouter, and later, because of limits and overloads, I moved to a list of free models with a fallback between them. For the prompt, I improved it over several tries: it asks for a fixed JSON answer (score, summary, flagged ingredients with severity and reason, beneficial ingredients) and it states the scoring rule explicitly, so that scores can be compared. Models sometimes wrap the JSON in extra formatting, so the code removes it. The scores still change a little from one run to another.

**The diagrams.** The diagrams of the [Design](../03-design/) section were written as text (Mermaid and Graphviz) and turned into images. I compared each one with the real code and asked for changes until they matched. The architecture diagram, for example, was drawn again because the first version was not clear.

**Building the application step by step.** I did not write everything at once. The order of the work, visible in the commit history, was: the client of Open Beauty Facts, the AI analysis, the SQLite storage, the Flask application with the main page, the analysis route, the history route, the manual-ingredients fallback, the photo upload (interface first, then the reading of the image), and finally cleaning, tests and the first release. Later versions added the versioned API, the score out of 100 and the handling of AI failures.

**Testing and fixing errors.** I tried the application with real products and real photos, and most of the important fixes came from these tries:

- Some products have ingredients only in a language-specific field, or no name; the barcode client was changed to handle both.
- The error "AI analysis temporarily unavailable" turned out to have two causes: the free provider was overloaded (it answers "success" with an error inside), and in one case I was running an old copy of the server that did not have my fix. I added retries and the fallback to other models, and restarted the server.
- A photo failed because the file was not really a JPEG; the code now detects the real image type.

The automated tests were written for the behaviours I had to protect after each fix, and the manual tests are listed in [Validation](../05-validation/).

**Release problems and what they taught me.** The release part gave me the most trouble, and also the most lessons.

- The first release in the pipeline failed with the message "No GitHub token specified". Reading the log carefully showed that the pipeline needs a token stored as a secret in the repository settings, and that a secret is not inherited from my own account. After creating the secrets, the release worked. Since then I read the error text of a failed run before trying anything else.
- Tests that passed on my computer failed on the pipeline, because the AI client was created at start-up and needed an API key, which the pipeline does not have. I changed the code so that the client is created only when it is used. The lesson was that tests must not depend on my local configuration.
- The test matrix failed on Python 3.9, because the dependencies need Python 3.10 or newer. I declared the supported versions in `pyproject.toml` and removed 3.9 from the matrix.
- The package had to be renamed before it could be published on PyPI, and the organization of the project was renamed as well, so links and names had to be corrected in several places. I learned that names of packages and organizations should be decided before they spread over the files.
- I also learned how much the commit messages matter: the version number is calculated from them, so a wrong type (`fix` instead of `feat`) would give a wrong release.

After solving these problems, the releases (1.0.0 to 1.2.1) were produced by the pipeline alone, and I understand much better why each step of the pipeline exists.

## How I used the AI assistant

I used Claude as a coding assistant to work faster. I wrote some parts of the code myself and asked Claude to check them, and the corrections were then made together with it; other parts were developed in a dialogue, where I asked for explanations of what each step did. I also asked it whether my tests were complete and whether I was reading the results correctly, for example the coverage report. The drafts of this report were prepared with its help too.

The decisions, the checking of the results (running the tests, trying the application with real barcodes and photos, comparing the text of the report with the real code and configuration) and the final choice of what was kept were made by me. The details of this use are given at the beginning of the report.

## Strengths

The application works from beginning to end. I can give it a barcode, a typed list or a photo, and I get a score out of 100, a short explanation and the ingredients to watch. If a barcode is not found, the page asks for the ingredients and does not simply stop. I am glad about this, because it came from my own experience of the first tries.

I am also happy with how it behaves when the AI is not available. Free models are often busy, and at the beginning this made the application fail many times. Now it tries again, changes model, and only then shows a clear message.

The structure of the code is simple: each outside service has its own file, and the Flask file only coordinates them. This made it easy to change the AI models and to write tests without internet.

On the engineering side, I am satisfied with the 24 tests, the 95% coverage and the pipeline, which checks the code on three operating systems and four Python versions at every push. The four releases on PyPI and GitHub were made automatically from the commit messages. I also tried to be honest in the documentation: the README and this report say that the result comes from an AI model, can be wrong and is not medical advice.

## Weaknesses

| Area | Weakness | Possible improvement |
|------|----------|----------------------|
| AI reliability | Free models are slow (up to about two minutes) and sometimes unavailable, so an analysis can fail for reasons outside the code. The list of free models also changes over time. | A paid or personal provider key. |
| Score consistency | The scoring rule is written in the prompt and applied by the AI model, not computed by the code. The same list can get slightly different scores, and the score cannot be checked or reproduced. | Let the model only classify the ingredients and calculate the score in the code. |
| Original plan not completed | My project proposal included a local layer that saves analysed products to avoid repeated calls and lets users go back to past analyses. Only a part was done: every analysis is saved in SQLite, but the saved data are never reused to answer a repeated request (the barcode is searched again and the AI is called again), and the history cannot be seen on the page, only through the `/api/v1/history` address. | Look in the database before calling the outside services; add a history view to the page. |
| Data quality | Open Beauty Facts is maintained by its community: many products or ingredient lists are missing. | More data sources; adding the missing data to Open Beauty Facts. |
| Photos | Reading the label depends on the vision model. A photo that cannot be read does not always give an empty result: the model may answer with text that is not a real ingredient list, so the "unreadable photo" error (422) is hard to trigger. | Check that the extracted text looks like an ingredient list before analysing it. |
| Stored data | The lists of flagged and beneficial ingredients are saved as the text form of a Python list, not as real JSON. | Save valid JSON, with a migration for existing databases. |
| User interface | The page has no visible message that the result is AI-generated (it is only in the README). The page was checked by hand and not with automated tests. | Add a visible disclaimer and browser-level tests. |
| Security and deployment | No user accounts or request limits; the application runs with the Flask development server, so it is not suitable for a public server. | Authentication, a production server and limits (see [Future works](../12-future/)). |
| Testing | The functions of `database.py` that really write and read SQLite are not run by the automated tests, because the tests replace them with test doubles (coverage of this module is 74%). I checked them by hand, since saving and the history route work in the real application, but a change there would not be caught automatically. There are also no automated system tests with the real services. | Tests with a temporary database; a few system tests run on demand. |
| Process | I worked on one branch, without pull requests, issues or code review, and wrote the tests after the code instead of before it. | Short-lived branches with pull requests; tests first for new features. |
| Dependencies | GitHub reports known vulnerabilities in dependencies, which I did not review or fix during the project. | Review the alerts and update the dependencies regularly. |
| Naming | The variable that holds the OpenRouter key is still called `GEMINI_API_KEY`, which is misleading. | Rename it and keep the old name working for some time. |

## Overall assessment

The product reaches its purpose as an educational project: it works with real products, handles the main failures, and is built, tested and released in a professional way. Its limits come mostly from the choices I made to keep it free and simple, namely free AI services and a scoring rule left to the model, and from one part of my first plan that remained incomplete. The main lesson for me is about process. Working alone made things quick to organise, but without branches, reviews and tests written first, many problems were found by trying the application and not by preventing them.
