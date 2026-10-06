---
title: Validation
has_children: false
nav_order: 6
---

# Validation

## Testing approach

The software was tested with automated tests written in **pytest**, plus manual acceptance tests of the web page. pytest was chosen because it was already set up in the project template, tests are plain functions with ordinary `assert` statements, and it works with `unittest.mock` for replacing outside services.

Test-driven development was **not** followed. The code of each feature was written first and the tests came afterwards (the commit `test: replace template test with app route tests` is an example). The one exception is the retry and fallback logic for the AI service: each failure found in real use (overloaded provider, empty answer) was fixed together with a test that reproduces it.

Because the application depends on two outside services (Open Beauty Facts and an AI service reached through OpenRouter), **no automated test calls the real services**. They are slow, need an API key, can fail for reasons unrelated to the code, and the free AI models give different answers at each call. All outside calls are replaced by test doubles; the real services are exercised only in the manual tests.

The tests run automatically on every push (see [CI/CD](../08-cicd/)): first on Ubuntu with static checks and coverage, then on Ubuntu, Windows and macOS with Python 3.10, 3.11, 3.12 and 3.13 (12 combinations).

## Testing (automated)

The suite has **24 tests, and all of them pass**. The tests are in the `tests` folder, one file per module of the application. The requirement each test checks is given in the tables below and, from the requirement side, in the acceptance-criteria table of [Requirements](../02-requirements/).

**Test doubles.** `unittest.mock.patch` replaces functions of the application and of the `requests` library for the time of one test.

- *Stubs:* a replaced function that only returns a prepared value, for example the product catalogue returning a fixed product, or the AI analysis returning a fixed score of 80.
- *Fakes of failure:* a replaced function that raises an error (`side_effect`), for example a network error or an AI model that keeps failing.
- *Mocks:* a replaced function whose use is then checked, for example that a successful analysis calls `save_analysis` exactly once.
- `time.sleep` is replaced as well, so the retry tests do not wait.

Stubs are used when only the answer matters, mocks when the call itself is the thing to verify.

### Unit testing

The unit tests check each client module alone. The outside world is replaced, so a test checks only the logic of the module.

**`tests/test_beauty_api.py` (6 tests): product catalogue client.** The `requests.get` call is replaced.

| Test | What it checks | Requirement |
|------|----------------|-------------|
| `test_product_found_with_ingredients` | A product record is turned into name, brand and ingredient text. | FR1, FR2 |
| `test_unknown_barcode_returns_none` | An unknown barcode gives no product. | FR3 |
| `test_missing_name_and_brand_use_defaults` | A missing name or brand becomes "Unknown Product" / "Unknown Brand". | FR2 |
| `test_falls_back_to_language_specific_ingredients` | If the general ingredient text is empty, a language-specific text is used. | FR2 |
| `test_product_without_ingredients_returns_empty_text` | A product without ingredients gives an empty text, so it is treated as not usable. | FR3 |
| `test_network_error_returns_none` | A network error behaves like an unknown barcode instead of crashing. | FR3, NFR3 |

**`tests/test_analyzer.py` (8 tests): AI client.** The AI client object is replaced by a fake that returns prepared answers.

| Test | What it checks | Requirement |
|------|----------------|-------------|
| `test_analyze_parses_json_wrapped_in_markdown` | An answer wrapped in a code block is still read correctly. | FR7-FR10 |
| `test_analyze_retries_when_provider_is_overloaded` | An "overloaded" answer is repeated and then succeeds. | NFR3 |
| `test_analyze_retries_on_empty_content` | An empty answer is repeated. | NFR3 |
| `test_analyze_falls_back_to_next_model` | When one model keeps failing, the next one in the list is tried. | NFR3 |
| `test_analyze_gives_up_after_all_models_fail` | If every model fails, a clear error is raised. | NFR3, FR13 |
| `test_extract_ingredients_from_image_returns_clean_text` | The text read from a photo is returned cleaned. | FR5 |
| `test_extract_ingredients_raises_when_all_vision_models_fail` | If no image model works, a clear error is raised. | NFR3 |
| `test_detect_mime_type` | PNG, WebP and JPEG images are recognised from their first bytes. | FR5 |

**Result:** 14 of 14 unit tests pass. Coverage of the modules tested: `beauty_api.py` 100%, `analyzer.py` 96%.

### Integration testing

**`tests/test_app.py` (10 tests): the API layer with the web framework.** These tests send requests to the application through Flask's test client, so routing, input checking, JSON answers and status codes run for real. The three modules of the lowest layer (`get_product_by_barcode`, `analyze_ingredients`, `extract_ingredients_from_image`) and the database functions (`save_analysis`, `get_history`) are replaced by stubs or mocks. What is tested is therefore the coordination done by the API layer: which step comes next, what is returned, and which error is reported.

| Test | What it checks | Requirement |
|------|----------------|-------------|
| `test_index_returns_page` | The home address returns the web page. | FR16 |
| `test_analyze_without_input_returns_400` | A request with no barcode and no ingredients is refused. | FR12 |
| `test_analyze_manual_ingredients` | Typed ingredients give a score, product name "Manual Entry", and the analysis is saved once (mock). | FR4, FR7-FR10, FR14 |
| `test_analyze_barcode_found` | A known barcode gives the product name, brand and a score. | FR1, FR2 |
| `test_analyze_barcode_not_found_returns_404` | An unknown barcode gives `not_found`. | FR3 |
| `test_analyze_ai_failure_returns_503` | A failing AI service gives `ai_failed`. | FR13 |
| `test_analyze_photo_without_file_returns_400` | A photo request without a file is refused. | FR12 |
| `test_analyze_photo_success` | A photo gives a score and product name "Photo Entry". | FR5 |
| `test_analyze_photo_unreadable_returns_422` | A photo from which nothing can be read gives an error. | FR6 |
| `test_history_returns_saved_analyses` | The history route returns the saved analyses as JSON. | FR14 |

All routes are called under `/api/v1`, so the tests also confirm the versioned address (FR15).

**Result:** 10 of 10 integration tests pass. Coverage of `app.py`: 90%.

**What is not covered.** No automated test connects two *real* modules, for example the API layer with the real database. As a result, `database.py` is the least covered module (74%): its functions that really write to and read from SQLite are only run indirectly. This is a known gap, listed in [Self-evaluation](../11-selfevaluation/).

### System testing

There are **no automated system tests**: nothing starts the real server and calls the real Open Beauty Facts and AI services. The reasons are given above (key needed, unreliable free models, different answers at each run). Containers were not used for testing either; the clean environment is provided by the CI machines, which start from an empty virtual machine at every run.

The system as a whole is verified by the manual acceptance tests below.

**Overall results of the automated suite** (run with `poe coverage` and `poe coverage-report`):

| Measure | Value |
|---------|-------|
| Tests passed | 24 of 24 |
| Total coverage | 95% |
| Coverage by module | `beauty_api.py` 100%, `analyzer.py` 96%, `app.py` 90%, `database.py` 74% |
| Required minimum (NFR8) | 70% |
| Other checks (ruff, mypy, format check) | pass |

## Acceptance tests (manual)

The manual tests run the whole system for real: real server, real Open Beauty Facts, real AI models. They were repeated before the final release (1.2.1). To repeat them, start the application as described in the [User guide](../09-user-guide/) with a valid key in `.env`, and open `http://127.0.0.1:5000`.

| # | Scenario | Steps | Expected result | Requirement | Result |
|---|----------|-------|-----------------|-------------|--------|
| 1 | Barcode of a known product | Enter the barcode of a product that has ingredients in Open Beauty Facts and press Analyze. | Name, brand, a score out of 100, a summary and ingredient lists appear. | FR1, FR2, FR7-FR10 | Passed |
| 2 | Unknown barcode | Enter a number that is not a product (for example `0000000000000`). | A message says the product was not found and asks for the ingredients. The input stays. | FR3 | Passed |
| 3 | Typed ingredients | Type a list such as `Aqua, Glycerin, Sodium Lauryl Sulfate, Parfum` and press Analyze. | A score and lists appear; the product name is "Manual Entry". | FR4, FR7-FR10 | Passed |
| 4 | Label photo | Upload a clear photo of an ingredient label (JPEG). | The list is read from the photo, then a score appears; the product name is "Photo Entry". | FR5 | Passed |
| 5 | Waiting feedback | Start any analysis and watch the page. | A progress bar and the elapsed seconds are shown until the result arrives. | NFR2 | Passed |
| 6 | Score colours | Obtain a low score (a list with many irritants), a medium one and a high one. | Below 40 the score is red, 40 to 69 orange, 70 or more green. | FR11 | Passed |
| 7 | History | Open `http://127.0.0.1:5000/api/v1/history` after several analyses. | The analyses are listed, newest first, with name, brand, score and summary. | FR14 | Passed |

**Success rate:** 7 of 7 scenarios passed in the final run.

Because the score and the lists come from an AI model, the exact numbers differ between runs and between models. The manual tests therefore judge whether the result is *present and sensible* (a score between 0 and 100, flagged ingredients that really appear on the list), not whether it equals a fixed value. Occasionally, when all free models are busy, a scenario fails with the "temporarily unavailable" message; repeating it a minute later normally works. This is a limitation of free AI services, not of the application logic (see [Self-evaluation](../11-selfevaluation/)).
