# Draft Structure

Create these files in the draft folder (default `docs/legal-compliance/`).

## README.md

```markdown
# Legal Compliance Draft: <Product name>

> Status: DRAFT for developer and lawyer review. No application code was changed.
> Generated: <date>. Laws checked on: <date>.

## Business profile
<Legal name, ID, country, address, contact, product summary, B2C/B2B, target countries, minors, payments, sector>

## Laws that apply
| Country / state | Law | Why it applies | Key duties | Source (checked <date>) |

## Checklist
| # | Item | Status | Notes |
| 1 | Terms and Conditions | Missing / Draft ready / Done | |
| 2 | Privacy Policy | | |
| 3 | Cookies Policy and consent decision | | Banner needed: yes/no, why |
| 4 | Register checkboxes with links | | |
| 5 | Consent records (when and how) | | |
| 6 | Refund Policy | | Period chosen and why |
| 7 | Data minimization | | Fields to remove |
| 8 | Media rights | | Assets to replace |
| 9 | Local law duties | | (database registration, DPO, complaints book, etc.) |

## High-risk gaps
<Ordered list, highest risk first>

## Decisions made
<Each choice made for the user, with the reason and the alternative. Example: one global refund period of 10 calendar days to meet Argentina and Chile.>

## Open questions and TODOs
<Each [TODO] from the documents>

## For the lawyer
<Specific points that need legal review>

## Ongoing duties
<Updates, re-consent, data requests, breach plan, re-check before new countries>
```

## documents/

One Markdown file per document, in each target language. Example for two languages: `privacy-policy.es.md` and `privacy-policy.en.md`. Use `[TODO: ...]` for unknown facts. Never invent facts.

## data-inventory.md

The table from `references/data-minimization.md`.

## media-inventory.md

The table from `references/media-rights.md`.

## implementation-plan.md

```markdown
# Implementation Plan

Stack: <detected stack>
Approve this plan before implementation.

## Step N: <Title>
- Why: <legal reason, one line>
- Files: <paths to create or change>
- Changes: <precise description>
- Code sketch: <short snippet in the project's language and framework>
- Data migration: <if any>
- Test: <how to verify, including the legal evidence it produces>
```

Usual steps, in order (skip the ones that do not apply):

1. Legal document storage with versions (database table, or versioned files plus a hash).
2. Legal pages (routes or screens) that render the current version, and footer and settings links.
3. Consent records table (append-only) and a service to write records.
4. Register form: two unchecked checkboxes with links, optional marketing checkbox, server-side validation, consent record in the same transaction. Repeat for every sign-up path.
5. Re-consent screen for existing users and for material changes.
6. Cookie handling: remove or block trackers before consent, consent banner if needed, GPC support, cookie settings link.
7. Checkout: refund policy link and acceptance before payment, withdrawal or cancel button where the law requires it.
8. Data minimization changes: remove or make optional the fields in `data-inventory.md`.
9. Data subject request flow: a form or email channel, and the internal process to answer within the deadline. Account deletion in the app (required by Apple and Google app stores).
10. Media replacements from `media-inventory.md`.
11. Admin export of consent records per user.
