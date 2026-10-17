---
name: legal-compliance
description: Legal compliance setup for any website, app, SaaS, or startup, in any tech stack. Covers Terms and Conditions, Privacy Policy, Cookies Policy and cookie consent, register-form consent checkboxes, traceable consent records (proof of when and how each user accepted), Refund policy, data minimization, image and media rights, and local law checks (Latin America, United States, and others). Interviews the user about the business, market, and tech, then writes a reviewable draft of legal documents and code changes before anything is applied. Use this skill whenever the user mentions terms of service, privacy policy, cookies, cookie banner, GDPR, LGPD, CCPA, Habeas Data, consent, refund or return policy, "derecho de retracto", legal pages, launching or going live, preparing a startup or MVP for production, or asks if a project is "legal" or "compliant", even if they do not say "compliance".
---

# Legal Compliance

Make a project legally ready to launch: legal documents, consent capture, consent evidence, cookie handling, refund rules, data minimization, media rights, and local law checks.

The output is a **draft for review**, not applied changes. A developer reads the draft, approves it, and only then an agent implements it. This keeps every legal and code decision visible and reversible.

## Important limits

You are not a lawyer, and laws change. Do these things in every run:

- Search the web to confirm the current state of each law you cite. Record the source URL and the date you checked it. The reference files in this skill can be out of date.
- Mark each legal claim you are not sure about with `(verify)`.
- Tell the user that a local lawyer must review the documents before launch, especially for regulated sectors (health, finance, minors, gambling, crypto, AI decisions about people).
- Never promise "full protection". The goal is to remove the common and expensive gaps.

## Workflow

### Step 1: Inspect the project first

Before you ask questions, learn what the code already tells you. This reduces the questions to the ones only the user can answer. Look for:

- Tech stack, frontend and backend, mobile or web, where the database is.
- Existing legal pages (search for "terms", "privacy", "cookies", "refund", "legal", "términos", "privacidad").
- Registration and login flows, and every form that collects personal data. List each field.
- Third-party scripts and SDKs: analytics, ads, pixels, session replay, chat widgets, maps, embedded videos, fonts from CDNs, error tracking, A/B testing.
- Cookies and local storage that the app sets.
- Payments: provider, one-time or subscription, digital or physical goods.
- Emails and SMS: transactional and marketing.
- Images, videos, fonts, icons, and audio in the repo or loaded from URLs.
- Hosting and data location (cloud region).
- Any AI features that process user data.

If there is no code yet, skip this step and ask more in Step 2.

### Step 2: Interview the user

Read `references/intake-questionnaire.md`. Ask only the questions that Step 1 did not answer. Ask in small groups (5 to 8 questions at a time), with the most important group first: legal entity and countries. Offer sensible default answers where possible so the user can answer fast.

Do not guess facts about the business. A wrong company name, country, or data use in a legal document is worse than a missing one. If the user does not know an answer, write a clear `[TODO: ...]` placeholder and add it to the open questions list.

### Step 3: Find the laws that apply

Laws apply based on where the business is established **and** where its users are. A Colombian startup that sells to Mexico and the US must follow parts of all three.

1. Build the list of countries (and US states) from the interview.
2. Read the matching file in `references/jurisdictions/`:
   - `latam.md` for Colombia, Mexico, Brazil, Argentina, Chile, Peru, and general Latin America notes.
   - `united-states.md` for federal and state laws.
   - `other.md` for any other country (EU and UK short notes, and a method to research an unknown country).
3. Search the web to confirm each law is still current and to find changes after the reference date.
4. Use the **strictest** applicable rule when rules conflict and a single global flow is simpler. For example, a single refund period that meets the longest legal minimum is easier than one per country. Tell the user when you make this choice and what it costs them.

### Step 4: Audit and design each item

Work through each item below. For each one, read the reference file it names.

| Item | Reference |
| --- | --- |
| Terms and Conditions | `references/documents.md` |
| Privacy Policy | `references/documents.md` |
| Cookies Policy and consent decision | `references/cookies.md` |
| Register form checkboxes with links | `references/consent-records.md` |
| Traceable consent records | `references/consent-records.md` |
| Refund Policy | `references/documents.md` (refund section) and the jurisdiction file |
| Data minimization | `references/data-minimization.md` |
| Image and media rights | `references/media-rights.md` |
| Local law checklist | the jurisdiction files |

Refund period rule: the user's default is **5 days after purchase**. Use the longer of this default and the legal minimum of each country that applies. Note if the law counts business days or calendar days, and from which event (purchase date, delivery date, or contract date).

### Step 5: Write the draft

Create the draft folder in the project. Default path: `docs/legal-compliance/`. Use the structure in `assets/draft-structure.md`. In short:

```
docs/legal-compliance/
├── README.md                  # Summary, business profile, laws, checklist, open questions
├── documents/
│   ├── terms-and-conditions.md
│   ├── privacy-policy.md
│   ├── cookies-policy.md
│   └── refund-policy.md
├── data-inventory.md          # Each data field, its purpose, legal basis, retention
├── media-inventory.md         # Each media asset, source, license, status
└── implementation-plan.md     # Code changes to make, step by step, in the project's stack
```

Write the legal documents in the language (or languages) of the target users. Use the real business data from the interview. Write in plain language: short sentences, clear headings. Plain language is easier for users to understand, and many laws (for example in Brazil, Mexico, and California) require clear and accessible notices.

Write `implementation-plan.md` for the project's actual stack, with file paths and code sketches, so an agent can implement it without more design work. If there is no code, describe the changes in stack-neutral terms (data model, UI behavior, API behavior).

### Step 6: Present and wait for approval

Give the user a short summary:

- Laws that apply and why.
- The checklist with status: done, missing, or needs a decision.
- High-risk gaps first (for example: trackers that load before consent, no consent evidence, media without a license).
- Open questions and `[TODO]` items.
- Items a lawyer must review.

Then stop. Do not change application code until the user approves the plan. When they approve, implement `implementation-plan.md` step by step, and update the checklist in `README.md` as items are completed.

## Ongoing duties to tell the user about

Compliance is not a one-time task. Add these to the README:

- Update the documents when the product, data use, third parties, or law changes. Increase the version number, keep old versions, and ask users to accept again when a change is material.
- Keep consent records for as long as the law requires (see `references/consent-records.md`).
- Answer data subject requests (access, correction, deletion, opt-out) within the legal deadline.
- Have a data breach response plan with the deadlines of each country.
- Check again before entering a new country or adding a new data use.
