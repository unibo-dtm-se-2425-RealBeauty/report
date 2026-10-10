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
| Visible disclaimer | That the result is AI-generated and is not medical advice is written in the README and in this report, but not on the page where users read the result. |
| User accounts and limits | The application has no login and no limit on the number of requests, so it can only be used safely on a personal computer. |
| Public deployment | There is no production server, container image or hosted version. The application starts with Flask's development server. |
| Personal profile | The score and the comments are the same for everyone. Users cannot say, for example, that they have sensitive or dry skin, or that some substances cause them problems. |
| Comparison of products | Each result is seen alone. There is no way to put two or more analysed products side by side. |
| Automated checks of the page | The web page is checked by hand only. |

## What does not work as it should

- **Free AI models.** They can be slow (up to about two minutes) or temporarily unavailable. The application tries again and changes model, but if all are busy the user has to wait and retry. The free models available also change from time to time, so the list in the code must be updated.
- **Scores are not fully repeatable.** The scoring rule is written in the prompt and the model applies it. Since 1.3.0 a list that was already analysed always gets its saved result, but a new analysis of a very similar list (for example one letter different, as often happens with photos) can still get a very different score.
- **Saved results never expire.** If the prompt or the models change, the results saved earlier are still returned for the same list.
- **Unreadable photos.** The model may answer with some text even when the label cannot be read. The text is then analysed as if it were an ingredient list, and the "unreadable photo" error is rarely shown.
- **Products missing in Open Beauty Facts.** Many products, or their ingredient lists, are not in the database. The application then asks for the ingredients, which is correct, but it means the barcode does not always save time.
- **Old saved data.** Records saved before 1.3.0 keep the old format of the ingredient lists (Python text instead of JSON). The application reads both, but another program would read only the new records reliably. Changing the table also requires deleting the old database file, because there is no migration.
- **Misleading variable name.** The key of the AI service is read from `GEMINI_API_KEY`, even though it is now an OpenRouter key.
- **Dependency alerts.** GitHub reports known vulnerabilities in some dependencies, which were not reviewed during the project.
- **Less tested parts.** No automated test runs the API together with the real database (this was checked by hand), and there are no automated tests with the real services.
- **Deprecation warning.** The creation time is set with `datetime.utcnow()`, which recent Python versions mark as deprecated; the tests show a warning. It does not change the results today but will have to be replaced by a timezone-aware time.

## Future developments

The following are listed in the order in which they would be done.

1. **Make the score reproducible.** Ask the AI model only to classify each ingredient (severity and reason) and calculate the score in the code with the stated rule. The score becomes the same for the same classification, and it can be tested.
2. **Check the text read from a photo** before analysing it, for example by requiring that it looks like a list of ingredients, and show a clear message if it does not.
3. **Show the disclaimer on the page** next to every result.
4. **More reliable AI access.** Use a personal key, or a paid plan, to avoid the limits of free models; keep the fallback list.
5. **Clean-up of the technical debt.** Rename the key variable, review the dependency alerts, replace `datetime.utcnow()`, convert the old saved records to JSON, store the model and prompt version with each saved result so that old results can be refreshed, and add tests through the API with a temporary database and a few system tests that can be run on demand with the real services.
6. **Wider data.** Use more product databases, and contribute to Open Beauty Facts the ingredient lists that users type or photograph for products that were missing.
7. **Comparison of products.** Let users choose two or more products from the history and see them side by side: score, number of flagged ingredients by severity, the high-risk ingredients of each, and the ingredients they have in common. This would help the shopper of US9, who wants to choose between products already checked. It needs no new outside service, only a new view and a route that returns several saved analyses.
8. **Personal profile.** Let users create a profile with their skin type (for example dry, oily or sensitive) and the skin conditions or sensitivities they want to take into account, and include it in the prompt, so that the comments and the highlighted ingredients are specific to them. This is a larger change than it looks, because skin conditions are health data: under the EU General Data Protection Regulation (GDPR, Article 9) they are a special category of personal data. Storing them would require user accounts, explicit consent, the possibility to see and delete the profile, encrypted storage, and care about what is sent to the AI service. For this reason it was not done in this project, which has no accounts and stores no personal data.
9. **Use on a phone.** Read the barcode with the phone camera instead of typing it.
10. **Public deployment.** Add user accounts, request limits, a production web server with HTTPS and a container image, and publish a hosted version.
