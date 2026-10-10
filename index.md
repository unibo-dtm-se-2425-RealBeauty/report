---
title: Home
layout: home
has_children: false
nav_order: 1
---

# RealBeauty

*Know what's really inside your personal care products.*

### Authors

- [Serra Özkasap](mailto:serra.ozkasap@studio.unibo.it)

Project work for the Software Engineering course, University of Bologna (Digital Transformation Management).

## Abstract

RealBeauty is a small web application that helps people understand what is inside a cosmetic or personal care product. Reading an ingredient list is hard, and checking each name by hand takes time, so the application does this work and returns a safety score from 0 to 100, a short explanation, the ingredients that deserve attention (with a severity and a reason) and the ones that are beneficial.

The ingredients can be provided in three ways: by typing the product barcode, which is looked up in the open database Open Beauty Facts; by pasting the list of ingredients, which is also the fallback when a product is not in the database; or by uploading a photo of the label, from which an AI vision model reads the list. The analysis itself is done by an AI model reached through OpenRouter. Every completed analysis is saved in a local SQLite database; when the same ingredient list is given again, the saved result is returned at once instead of asking the AI again, and the past analyses can be reviewed on the page.

The system is a Flask application with a layered client-server architecture: a web page, an application layer that coordinates the steps and a lowest layer with one module for each outside service. The HTTP API is versioned (`/api/v1`). Because free AI models are often busy, the application retries and falls back to other models before reporting an error.

The software was verified with 33 automated tests (97% coverage) that replace the outside services with test doubles, and with manual acceptance tests on real products and photos. A GitHub Actions pipeline checks the code on three operating systems and four Python versions at every push and publishes new versions on PyPI and GitHub automatically from the commit messages; six releases were produced this way.

The report follows the structure of the course: concept, requirements, design, development, validation, release, deployment, CI/CD, user and developer guides, self-evaluation and future work. Its main limits are those of the free tools used: variable speed and availability of the AI models, scores that are not fully repeatable, and products missing from the database. RealBeauty is an educational project and its results are not medical advice.

## Disclaimer

During the preparation of this work, the author used Claude (Anthropic) to discuss design choices, to write and check code and tests, to create the diagrams, and to draft and revise the text of this report.

After using this tool/service, the author reviewed and edited the content as needed
and takes full responsibility for the content of the final report/artifact.

The application described in this report also uses AI models at run time (through OpenRouter) to analyse ingredients and to read label photos; this is part of the product and is described in the report.
