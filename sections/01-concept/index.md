---
title: Concept
has_children: false
nav_order: 2
---

# Concept

## Type of product

RealBeauty is a **web application** with a **web-service** behind it.

- People use it through a single web page, opened in an ordinary browser.
- The same functions are available to other programs through a versioned HTTP API (`/api/v1`), so the product can also be used as a web-service.

The software does not analyse ingredients by itself: it combines two external services (a public database of cosmetic products and an AI service) and adds the logic that turns their answers into a safety score that is easy to read.

## How it is used

1. The user gives the product in one of three ways: the **barcode**, the **ingredient list** typed or pasted, or a **photo** of the label.
2. If a barcode is given, the system looks the product up in Open Beauty Facts. If the product is missing, the user is invited to give the ingredients in another way.
3. If a photo is given, the system reads the ingredient list from the picture.
4. The ingredient list is judged by an AI model, which returns a score from 0 to 100, a short summary, the concerning ingredients (with severity and reason) and the beneficial ones.
5. The result is shown on the page and saved in the history.

## Use case collection

**Who are the users and where are they?**
The main users are consumers who buy personal care products (shampoo, creams, deodorants, make-up). They are often *in front of a shelf* in a shop, or at home with a product they already own. A second group is made of developers who want to call the analysis from their own programs. The roles are described in the [Requirements](../02-requirements/) section (shopper, sensitive user, integrator).

**When and how often do they interact with the system?**
Occasionally and in short sessions: typically when choosing or changing a product, a few times a week at most. A single analysis takes from a few seconds to about two minutes, because the AI models used are free and can be slow or busy. The user therefore has to wait for the answer, and the page shows the elapsed time while waiting.

**How do they interact, and with which devices?**
Through a web browser, on a computer or a phone. The page has a single-column layout and does not need an account or an installation. The barcode is typed (the user reads it under the bars); the label can be photographed with the phone camera and uploaded. Developers use any HTTP client.

**Does the system store users' data? Which data? Where?**
Yes, only a minimal history. For every analysis the system saves the barcode (if any), the product name, the brand, the score, the summary and the date and time, in a small database on the machine where the application runs. Uploaded photos are not stored. There are no user accounts, so the history is shared by everyone who uses that installation.

**Which external services does it depend on?**
- *Open Beauty Facts*: a free, community-maintained database, used to find the name, brand and ingredients of a product from its barcode. Many products are missing or have no ingredient list, which is why the manual and photo inputs exist.
- *OpenRouter*: a gateway to AI models. Only free models are used, so answers can be slow, and a model may be busy; the system therefore retries and falls back to other models.

**What the product is not.**
RealBeauty is an educational project. Its output comes from an AI model, may be wrong, and is not medical or dermatological advice.
