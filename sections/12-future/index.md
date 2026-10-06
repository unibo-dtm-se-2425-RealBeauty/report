---
title: Future work
has_children: false
nav_order: 13
---

# Known issues and future work

This section lists what is missing, what does not work as it should, and what could be done next. The same points are also discussed from the developer's point of view in [Self-evaluation](../11-selfevaluation/).

## What is missing

| Missing | Notes |
|---------|-------|
| Reuse of saved analyses | Every analysis is saved, but a repeated barcode is looked up and analysed again. A saved result could be returned immediately, without calling Open Beauty Facts and the AI. This was part of the original project idea. |
| History in the web page | The history exists only as the address `/api/v1/history`, which shows technical text (JSON). The page has no view for it and no way to open an old analysis. |
| Visible disclaimer | That the result is AI-generated and is not medical advice is written in the README and in this report, but not on the page where users read the result. |
| User accounts and limits | The application has no login and no limit on the number of requests, so it can only be used safely on a personal computer. |
| Public deployment | There is no production server, container image or hosted version. The application starts with Flask's development server. |
| Personal profile | The score is the same for everyone. Users cannot say, for example, that they are allergic to some substance or have sensitive skin. |
| Automated checks of the page | The web page is checked by hand only. |

## What does not work as it should

- **Free AI models.** They can be slow (up to about two minutes) or temporarily unavailable. The application tries again and changes model, but if all are busy the user has to wait and retry. The free models available also change from time to time, so the list in the code must be updated.
- **Scores are not fully repeatable.** The scoring rule is written in the prompt and the model applies it, so the same list may get slightly different scores in different runs.
- **Unreadable photos.** The model may answer with some text even when the label cannot be read. The text is then analysed as if it were an ingredient list, and the "unreadable photo" error is rarely shown.
- **Products missing in Open Beauty Facts.** Many products, or their ingredient lists, are not in the database. The application then asks for the ingredients, which is correct, but it means the barcode does not always save time.
- **Saved data format.** The lists of flagged and beneficial ingredients are saved as the text form of a Python list, not as valid JSON, so they cannot be read back reliably by another program. Changing the table also requires deleting the old database file, because there is no migration.
- **Misleading variable name.** The key of the AI service is read from `GEMINI_API_KEY`, even though it is now an OpenRouter key.
- **Dependency alerts.** GitHub reports known vulnerabilities in some dependencies, which were not reviewed during the project.
- **Less tested parts.** The functions of `database.py` that write and read SQLite are not run by the automated tests (they were checked by hand), and there are no automated tests with the real services.

## Future developments

The following are listed in the order in which they would be done.

1. **Reuse saved analyses and show the history.** Look in the database before calling outside services, and add a history list to the page. Together they complete the original idea of the local storage, and save time and calls. This change needs the lists to be saved as valid JSON first.
2. **Make the score reproducible.** Ask the AI model only to classify each ingredient (severity and reason) and calculate the score in the code with the stated rule. The score becomes the same for the same classification, and it can be tested.
3. **Check the text read from a photo** before analysing it, for example by requiring that it looks like a list of ingredients, and show a clear message if it does not.
4. **Show the disclaimer on the page** next to every result.
5. **More reliable AI access.** Use a personal key, or a paid plan, to avoid the limits of free models; keep the fallback list.
6. **Clean-up of the technical debt.** Rename the key variable, review the dependency alerts, add tests that use a temporary database, and add a few system tests that can be run on demand with the real services.
7. **Wider data.** Use more product databases, and contribute to Open Beauty Facts the ingredient lists that users type or photograph for products that were missing.
8. **Personalisation.** Let users record allergies or skin type, and mark the ingredients that matter for them; compare two products side by side.
9. **Use on a phone.** Read the barcode with the phone camera instead of typing it.
10. **Public deployment.** Add user accounts, request limits, a production web server with HTTPS and a container image, and publish a hosted version.
