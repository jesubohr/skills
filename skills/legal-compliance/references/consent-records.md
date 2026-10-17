# Consent Capture and Consent Records

Regulators and courts ask two questions: "Did this user agree?" and "To exactly what text?" The business must answer both with evidence. In many countries (for example Colombia, Mexico, Brazil, Peru) the burden of proof is on the business, not on the user.

## Register form

Add two separate checkboxes:

```
[ ] I accept the Terms and Conditions          -> link to /terms
[ ] I accept the Privacy Policy and authorize   -> link to /privacy
    the processing of my personal data
```

Rules:

- **Not pre-checked.** A pre-checked box is not valid consent in most jurisdictions.
- **Separate boxes.** One for terms (a contract) and one for privacy (data processing). Some laws require that data consent is not bundled with other terms.
- **Links open the exact current version.** Open in a new tab or a modal, so the user does not lose the form data.
- **The submit button stays disabled, or the server rejects the request, until both are checked.** Validate on the server too. Client-only validation is not evidence.
- **Optional consents get their own unchecked boxes.** For example marketing emails, SMS, or sharing data with partners. The user must be able to register without them.
- **Social login and other sign-up paths** (Google, Apple, magic link, invite link, mobile app) need the same checkboxes or an equivalent step before the account is active. Check every path that creates an account.
- **Users who registered before this change** must accept on their next login (a blocking accept screen).

The checkbox label text must also be versioned and stored, because it is part of what the user agreed to.

For US users, clickwrap (an unchecked checkbox or a clear "By clicking Create account, you agree to..." next to the button, with visible links) is enforceable. Browsewrap (a link in the footer only) is often not enforceable. The two-checkbox design satisfies both Latin America and the US.

## Legal document versions

Store every published version of each document:

| Field | Notes |
| --- | --- |
| `id` | Primary key |
| `document_type` | `terms`, `privacy`, `cookies`, `refund`, `marketing` |
| `version` | For example `2026-10-16` or `1.2.0` |
| `language` | For example `es-CO`, `en-US` |
| `content` | Full text, or a path to an immutable file |
| `content_hash` | SHA-256 of the content. Proves the text did not change later |
| `effective_at` | Date the version starts to apply |
| `is_material_change` | If true, users must accept again |

Never edit a published version. Publish a new version.

## Consent records

One row per user, per document, per action. **Append-only**: never update or delete rows during the retention period. A withdrawal is a new row, not a change of an old row.

| Field | Why |
| --- | --- |
| `id` | Primary key |
| `user_id` | Who. Keep the record even if the user account is deleted (see Retention) |
| `document_type` and `document_version_id` | To what exact text |
| `action` | `accepted`, `withdrawn`, `rejected` |
| `accepted_at` | When. Use **server time in UTC**, not client time |
| `method` | How. For example `register_checkbox`, `reconsent_modal`, `cookie_banner`, `checkout_checkbox` |
| `checkbox_label` | The exact label text the user saw |
| `source` | `web`, `ios`, `android`, `api` and app version |
| `page_url` or screen name | Where |
| `ip_address` | Supporting evidence. Note: IP is personal data. Mention it in the Privacy Policy |
| `user_agent` | Supporting evidence |
| `locale` | Language of the text shown |

Optional but useful: a request ID that links to server logs.

### Evidence quality

- Write the record in the same database transaction that creates the account. If the record fails, the account must not be created.
- Restrict write and delete access to the table. Consider a database role that can only insert.
- Back up the table with the rest of the data.
- Make an admin or script export: "show all consent records for user X" as CSV or PDF. A regulator request then takes minutes, not days.

### Retention

Keep consent records for the duration of the relationship plus the legal limitation period for claims. Check the jurisdiction files. A safe default when unsure is 5 years after the account ends, but confirm with the local law and a lawyer. When a user asks for deletion, you can usually keep the minimum consent evidence because a legal duty requires it. Say this in the Privacy Policy.

## Cookie consent records

Store cookie banner choices with the same model: categories accepted (`necessary`, `analytics`, `marketing`, `preferences`), banner version, timestamp, and an anonymous consent ID (for visitors without an account). Link the consent ID to the user when they log in.

## Re-consent flow

When a document version has `is_material_change = true`:

1. On the next login or app open, show a screen with a summary of the changes and a link to the full text.
2. Block use of the affected features until the user accepts, or let the user close the account.
3. Write a new consent record.
4. Also notify users by email before the effective date when the law or the terms require it.

For non-material changes (typos, contact info), publish the new version and show a "last updated" date. No new consent is needed.
