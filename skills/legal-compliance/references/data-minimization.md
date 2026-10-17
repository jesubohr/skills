# Data Minimization

Collect only the data the product needs to work. Every extra field adds legal duties, breach impact, and form friction. Most data protection laws state this principle directly (for example "finalidad", "necesidad", "proporcionalidad" in Latin America, and "data minimization" in US state laws).

## Method

1. List every personal data field the product collects: forms, sign-up, checkout, profile, analytics events, logs, third-party SDKs, support tools.
2. For each field, write the purpose in one sentence. If the team cannot write a clear purpose, recommend removing the field.
3. Mark each field:
   - **Required**: the service cannot work without it (for example email for login).
   - **Optional**: helps the user but is not needed. Make it optional in the UI and label it.
   - **Remove**: no clear purpose.
4. Mark sensitive data. Sensitive data needs stronger justification, explicit consent in most countries, and stronger security.
5. Set a retention period for each field and what happens at the end (delete or anonymize).
6. Write the result to `data-inventory.md`.

## Common over-collection patterns

| Pattern | Better option |
| --- | --- |
| Full birth date to check age | An "I am 18 or older" checkbox, or only the year, if no exact age is legally needed |
| Phone number required for an email-based product | Optional, or remove |
| Full address for digital products | Country only (needed for tax), or billing data held by the payment provider |
| Government ID number "just in case" | Do not collect unless a law or KYC rule requires it |
| Gender as a required field | Optional or remove |
| Storing card numbers | Use the payment provider tokens. Never store card data (PCI DSS) |
| Exact GPS location | Approximate location, or only when the feature is in use |
| IP addresses in logs forever | Rotate logs, truncate IPs, set a retention period |
| Personal data sent to analytics events | Send IDs, not emails or names |
| Copies of user data in AI prompts | Send the minimum, check the AI provider data retention terms |

## data-inventory.md format

| Field | Where collected | Purpose | Required? | Sensitive? | Legal basis | Shared with | Retention | Recommendation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

The Privacy Policy must match this table. If the code collects something the policy does not mention, the policy is false, and that is a violation by itself.
