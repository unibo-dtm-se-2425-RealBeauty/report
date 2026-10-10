---
title: Requirements
has_children: false
nav_order: 3
---

# Requirements

This section states **what** RealBeauty must do, not how it does it. How the requirements are realised is described in the [Design](../03-design/) section, and how they are checked is described in the [Validation](../05-validation/) section.

## Personas and user stories

Three kinds of users are considered.

- **Shopper**: a person standing in a shop (or at home) who holds a personal care product and wants a quick opinion before buying or using it. They use a phone or a laptop, and rarely read the small print on the label.
- **Sensitive user**: a person with sensitive skin, allergies or specific concerns, who wants to know *which* ingredients may be a problem and *why*.
- **Integrator**: a developer who wants to reuse the analysis from another program through a stable HTTP interface.

| ID | As a... | I want to... | So that... |
|----|---------|--------------|------------|
| US1 | Shopper | type or scan the barcode of a product | I do not have to read and type the ingredients myself |
| US2 | Shopper | paste the ingredient list by hand | I can still get an analysis when the product is not in the database |
| US3 | Shopper | upload a photo of the ingredient list on the label | I can analyse a product without typing anything |
| US4 | Shopper | see one score from 0 to 100, with a colour | I understand at a glance whether the product looks safe |
| US5 | Sensitive user | see which ingredients are concerning, with a severity and a reason | I can decide whether the product is right for me |
| US6 | Sensitive user | see which ingredients are beneficial | I get a balanced picture, not only warnings |
| US7 | Shopper | read a short summary in plain language | I do not need to know cosmetic chemistry |
| US8 | Shopper | get a clear message when something goes wrong | I know whether to retry, or to enter the data in another way |
| US9 | Shopper | review my previous analyses | I can compare products I have already checked |
| US10 | Integrator | call the analysis through a versioned HTTP API | my program keeps working when the service evolves |
| US11 | Shopper | get the same result again, at once, when I check a product I have already analysed | the score does not change by chance and I do not have to wait again |

## Requirements analysis

### Functional requirements

| ID | Requirement | Stories |
|----|-------------|---------|
| FR1 | The system shall accept a product barcode as input. | US1 |
| FR2 | Given a barcode, the system shall retrieve the product name, the brand and the ingredient list from a public product database. | US1 |
| FR3 | If the barcode is unknown, or the product has no ingredient list, the system shall tell the user and invite them to enter the ingredients manually. | US1, US2, US8 |
| FR4 | The system shall accept an ingredient list written as free text. | US2 |
| FR5 | The system shall accept a photo of a product label and extract the ingredient list from it. | US3 |
| FR6 | If no ingredient list can be read from the photo, the system shall tell the user. | US3, US8 |
| FR7 | The system shall assign each analysed product a safety score between 0 and 100, computed with a documented rule: start from 100, subtract 20 for each high-severity ingredient, 10 for each medium one and 3 for each low one, add 2 for each beneficial ingredient, and keep the result between 0 and 100. | US4 |
| FR8 | The system shall list the concerning ingredients, each with a severity (high, medium or low) and a short reason. | US5 |
| FR9 | The system shall list the beneficial ingredients. | US6 |
| FR10 | The system shall provide a summary of two or three sentences in plain language. | US7 |
| FR11 | The system shall show the score in a colour that reflects it: good (70 or more), medium (40 to 69), poor (below 40). | US4 |
| FR12 | The system shall reject a request that contains no barcode, no ingredients and no photo, explaining what is missing. | US8 |
| FR13 | If the analysis cannot be completed because the AI service is unavailable, the system shall tell the user to try again later. | US8 |
| FR14 | The system shall save every completed analysis (barcode if any, product name, brand, the ingredient list analysed, score, summary, flagged and beneficial ingredients, date and time) and allow the saved analyses to be listed, most recent first, with each ingredient list shown only once (its newest result). | US9 |
| FR15 | The system shall expose all its functions through an HTTP API whose address contains a version number. | US10 |
| FR16 | The system shall offer a single web page from which a user can run an analysis in any of the three ways (FR1-FR13) and review the saved analyses (FR14): a compact list showing score, product name and input method, where an opened row gives a short summary and a button shows the full result. | US1-US9 |
| FR17 | If the same ingredient list was analysed before (ignoring upper and lower case and extra spaces), the system shall return the saved result instead of running a new AI analysis, and shall tell the user that the result is a saved one. | US11 |

### Non-functional requirements

| ID | Requirement |
|----|-------------|
| NFR1 | **Usability.** An end user needs no account and no installation besides a web browser. |
| NFR2 | **Feedback.** While an analysis is running, the page shall show that work is in progress and how much time has passed, because the analysis can take up to two minutes. |
| NFR3 | **Robustness.** A temporary failure or overload of the AI service shall not stop the application; the system shall try again, and try alternative models, before giving up. |
| NFR4 | **Privacy.** The system shall not store the photos uploaded by users; only the ingredient text read from them may be kept. |
| NFR5 | **Security.** The secret key used to access the AI service shall never be part of the source code or of the public repository. |
| NFR6 | **Transparency.** The documentation shall state that the results are generated by an AI model and are not medical advice. |
| NFR7 | **Portability.** The software shall run on Linux, Windows and macOS, with Python 3.10 or newer. |
| NFR8 | **Maintainability.** The software shall have automated tests covering at least 70% of the code, and shall pass automatic style and type checks. |
| NFR9 | **Traceability.** Every release shall be built, tested and published automatically, with a version number derived from the commit history. |

### Implementation requirements

| ID | Requirement | Justification |
|----|-------------|---------------|
| IR1 | The software shall be written in Python, managed with Poetry. | The course provides a Python project template, with its tooling, for the project work. |
| IR2 | Product data shall come from [Open Beauty Facts](https://world.openbeautyfacts.org/). | It is free and open, and the project has no budget for paid data. |
| IR3 | The AI analysis shall use models that can be used at no cost. | The project has no budget; models are therefore expected to be slower and less reliable than paid ones (see NFR3). |
| IR4 | The source code shall be hosted on GitHub, with Conventional Commits, GitHub Actions and semantic-release. | They are required by the course for the project work. |

### Glossary

| Term | Meaning |
|------|---------|
| Barcode | The number printed under the bars on a product (typically EAN-13), identifying it worldwide. |
| Ingredient list | The list of substances in a cosmetic product, written on the label, usually with international (INCI) names, in decreasing order of quantity. |
| Open Beauty Facts | A free, community-maintained database of cosmetic products. Entries are created by volunteers, so some are missing or incomplete. |
| Safety score | A number from 0 (worst) to 100 (best) summarising how safe the ingredient list looks, computed by the rule in FR7. |
| Severity | How concerning an ingredient is: high, medium or low. |
| Beneficial ingredient | An ingredient the analysis considers good for skin or hair. In the interface these are called "safe highlights". |
| AI model | A language model, reached through the OpenRouter service, that reads an ingredient list and judges it, or reads a label photo and extracts the list. |
| Analysis | One complete run of the system on one product, producing a score, a summary and ingredient lists. |
| Saved result | An analysis read back from the local database instead of being produced again by the AI model (see FR17). In the code this is called the cache. |

## Acceptance criteria

Each criterion says how it is decided that the requirement is met. The last column says how it is checked: by an automated test (name of the test, see [Validation](../05-validation/)) or by a manual test.

| Req. | Acceptance criterion | Checked by |
|------|----------------------|------------|
| FR1, FR2 | Given a barcode that exists in the database and has ingredients, the response contains the product name, the brand and a score. | `test_analyze_barcode_found`, `test_product_found_with_ingredients`; manual test with real barcodes |
| FR2 | If the database has no general ingredient text but has one in a specific language, that text is used. | `test_falls_back_to_language_specific_ingredients` |
| FR2 | If the database entry has no name or brand, the response uses a default name and brand. | `test_missing_name_and_brand_use_defaults` |
| FR3 | Given an unknown barcode, the response is "not found" and the page asks for the ingredients. | `test_analyze_barcode_not_found_returns_404`, `test_unknown_barcode_returns_none`; manual test |
| FR3 | A product with an empty ingredient list is treated as not usable. | `test_product_without_ingredients_returns_empty_text`; manual test |
| FR3, NFR3 | If the product database cannot be reached, the system behaves as if the barcode were unknown. | `test_network_error_returns_none` |
| FR4 | Given ingredients typed by the user, the response has the score and the product name "Manual Entry". | `test_analyze_manual_ingredients` |
| FR5 | Given a photo with a readable label, the response has the score and the product name "Photo Entry". | `test_analyze_photo_success`; manual test with real photos |
| FR6 | Given a photo from which nothing can be read, the response is an error explaining that. | `test_analyze_photo_unreadable_returns_422` |
| FR7-FR10 | The response contains a score between 0 and 100, a summary, the flagged ingredients with severity and reason, and the beneficial ingredients. | `test_analyze_manual_ingredients`; manual test (the content itself is produced by the AI model and is judged manually) |
| FR7-FR10 | If the AI answer is wrapped in extra formatting, the data are still read correctly. | `test_analyze_parses_json_wrapped_in_markdown` |
| FR11 | A score of 70 or more is shown in green, 40 to 69 in orange, below 40 in red. | manual test |
| FR12 | A request without barcode and ingredients gets an error, as does a photo request without a file. | `test_analyze_without_input_returns_400`, `test_analyze_photo_without_file_returns_400` |
| FR13 | If the AI service fails, the response says it is temporarily unavailable. | `test_analyze_ai_failure_returns_503` |
| FR14 | After an analysis, the history lists it with product name, brand, score, summary, flagged and beneficial ingredients and input method, newest first. | `test_history_returns_saved_analyses`, `test_history_shows_method_for_photo_and_manual`; manual test (ordering) |
| FR14 | If the same ingredient list was saved more than once, the history shows only the newest result. | `test_history_shows_each_product_once` |
| FR15 | All functions are reachable under the `/api/v1` address. | all API tests in `test_app.py` |
| FR16 | The home address returns the web page. | `test_index_returns_page`; manual test |
| FR16 | The page lists the saved analyses as rows; only one row is open at a time, and "Show details" shows the full result at the top of the page. | manual test |
| FR17 | When a saved result exists, it is returned marked as saved, no AI analysis is run and no new record is added; a new analysis is marked as not saved. | `test_analyze_reuses_cached_result`, `test_analyze_new_result_is_not_cached` |
| FR17 | A saved result is found even when the list is typed with different upper and lower case or spacing, also for records saved by older versions; nothing is found for a new list. | `test_find_cached_ignores_case_and_spaces`, `test_find_cached_reads_rows_in_old_format`, `test_find_cached_returns_none_when_not_saved` |
| NFR2 | During an analysis the page shows a progress bar and the elapsed seconds. | manual test |
| NFR3 | If the AI service is overloaded or answers with nothing, the request is repeated; if it keeps failing, another model is tried; only then an error is reported. | `test_analyze_retries_when_provider_is_overloaded`, `test_analyze_retries_on_empty_content`, `test_analyze_falls_back_to_next_model`, `test_analyze_gives_up_after_all_models_fail`, `test_extract_ingredients_raises_when_all_vision_models_fail` |
| NFR4 | The saved data of an analysis contain no image, only text. | inspection of the database model (see [Design](../03-design/)) |
| NFR5 | The repository contains no key; the key is read from an untracked `.env` file. | inspection of the repository |
| NFR6 | The README states that the output is AI-generated and not medical advice. | inspection of the README |
| NFR7 | The tests pass on Linux, Windows and macOS with Python 3.10, 3.11, 3.12 and 3.13. | CI matrix on GitHub Actions |
| NFR8 | Test coverage is at least 70%; ruff, mypy and the format check pass. | CI (coverage report and static checks) |
| NFR9 | A push of a `feat:` or `fix:` commit to the main branch produces a new GitHub release and a new package on PyPI. | releases 1.0.0 to 1.3.1 |
