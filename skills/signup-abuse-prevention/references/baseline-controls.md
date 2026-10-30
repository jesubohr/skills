# Baseline Controls

Sources reviewed on 2026-10-29. Each section links its source. Numbers marked "illustrative" are examples from this skill, not standards. Provider limits and prices can change, so check the linked pages before you rely on them.

## Contents

- Check order
- Alternate signup routes
- Trusted client IP
- Turnstile
- Honeypot field
- Email verification and pending records
- Email canonicalization and aliases
- Confirmation tokens
- Waitlist
- Field validation and injection
- Generic responses and enumeration
- Rate limits and 429
- Cleanup

## Check order

Run the cheapest checks first. Run paid or slow checks last. Stop at the first failure.

1. Enforce a request-size limit and atomically count the request against the per-IP attempt limit.
2. Parse the body and validate field shape. This is local and free.
3. Check the honeypot field. This is local and free.
4. Verify the Turnstile token. This is one network call.
5. Check the per-recipient budget, the recipient-domain cap, the suppression list, and the global send cap.
6. Run optional paid lookups (IP intelligence, phone lookup).
7. Reserve the send atomically, write the pending record and an outbox row, then send the email or SMS from a worker. The worker rechecks suppression and caps at dispatch.

Never send an email or SMS before steps 1 to 5 pass. The same order applies to resend, magic links, password reset, waitlist, and invitations. See [email-reputation.md](email-reputation.md).

## Alternate signup routes

An attacker skips your form and calls the API. Protect the route, not the page.

- List every route that creates an account, joins a waitlist, or sends an email or SMS: the origin API, hosted auth endpoints, social login callbacks, magic links, invite links, resend, password reset, mobile apps, and admin or import scripts.
- Enforce equivalent server-side protections at each public entry point. Share logic where practical. Use route-specific controls for signed callbacks, trusted imports, and existing authenticated flows; do not add a browser widget requirement to them blindly.
- Hosted auth (for example Supabase Auth) exposes its own public endpoints. A frontend widget does not protect them. Turn on the provider's CAPTCHA enforcement and rate limits. Turn off anonymous sign-ins and unused providers. If you cannot enforce checks at the provider, disable public sign-up there and sign users up through your own backend.
  - Supabase CAPTCHA setup: https://supabase.com/docs/guides/auth/auth-captcha
  - Supabase rate limits (provider defaults, for example 30 requests per 5 minutes per IP on sign-up endpoints, and a per-project email cap): https://supabase.com/docs/guides/auth/rate-limits
- Test each public form-submission route directly without the widget token. Routes configured to require CAPTCHA must reject the request. Test signed callbacks and trusted routes against their own authentication requirements. Use a staging project.

## Trusted client IP

Rate limits need the real client IP. A spoofed header lets an attacker skip every limit.

- Trust a forwarding header only from a proxy you control. Count from the right side of `X-Forwarded-For`. Use a configured trusted-proxy count or list. Never use the leftmost value for security. If the server accepts direct internet traffic, no part of the header is trustworthy. Source (MDN): https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For
- Behind Cloudflare, read `CF-Connecting-IP`. Cloudflare recommends it over `X-Forwarded-For`. It is trustworthy only if the origin accepts traffic from Cloudflare alone. Use Authenticated Origin Pulls, Cloudflare Tunnel, or a firewall that allows only Cloudflare ranges. Sources: https://developers.cloudflare.com/fundamentals/reference/http-headers/ and https://developers.cloudflare.com/ssl/origin-configuration/authenticated-origin-pull/
- Other hosts set their own header. Read the host's documentation. Do not guess.
- Pass `remoteip` to Turnstile only when it is the trusted IP.

## Turnstile

Turnstile is a low-friction challenge. Keep the widget in its default mode and add friction only when data shows a need.

Sources: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/ and https://developers.cloudflare.com/turnstile/get-started/

- Server-side validation is mandatory. Cloudflare states this. A widget without a server check protects nothing.
- Send the token to `POST https://challenges.cloudflare.com/turnstile/v0/siteverify` with `secret` and `response`. The token is at most 2048 characters. It is valid for 300 seconds. It validates once. A reused or expired token returns `timeout-or-duplicate`.
- Check `success`. Also check `hostname` against your expected domain and `action` against the form's action name.
- Set an `action` on each widget (`data-action`), such as `signup` or `waitlist`. A token from another form then fails.
- Use a timeout near 10 seconds. Retry transient errors with the same `idempotency_key` (a UUID you create). Cloudflare's example uses 3 attempts with exponential backoff.
- Keep the secret key on the server. Use separate widgets for development, staging, and production. Limit hostnames per widget.
- Do not consume a token twice. If the auth provider verifies the token (for example, you pass `captchaToken` to Supabase), do not call `siteverify` yourself with the same token. Choose one verifier per request.
- On `expired` or `timeout-or-duplicate`, call `turnstile.reset` and ask the user to retry. Do not reuse tokens.
- Outage policy: see "Provider outages" in [tuning-and-operations.md](tuning-and-operations.md).
- The free plan limits (dated 2026-10-29): 20 widgets, 10 hostnames per widget. Source: https://developers.cloudflare.com/turnstile/plans/. Check the page again before you rely on these.

## Honeypot field

A honeypot is one extra form field that humans never see. Some bots fill it when they fill every field. Start with one CSS-hidden honeypot.

Build it so it stays accessible:

- Do not use `type="hidden"`. Many bots skip those fields.
- Hide it with CSS that moves it out of view, for example `position: absolute; left: -10000px;` on a wrapper. Avoid relying on `display: none` alone.
- Set `tabindex="-1"` on the input itself and `aria-hidden="true"` on its wrapper. A wrapper's tabindex does not remove its child input from keyboard focus. Test with a keyboard and screen reader. Reference for focus order: https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html
- Set `autocomplete="off"`. Browsers can ignore it (https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/autocomplete). So also use a neutral field name. Avoid names like `email`, `phone`, `address`, `url`, and `website`, which autofill may fill.
- Check the field on the server. First measure autofill false positives. Once enforcement is enabled, a filled field creates no record and sends no email. Return the same generic response as a success. Count the event; it is an abuse signal, not proof of a bot.

Fast completion is a weak signal. Record the time between form render and submit. A time under about 2 seconds (illustrative) can add to a risk score. Password managers and autofill finish forms quickly, so never block on this signal alone.

## Email verification and pending records

Source: https://cheatsheetseries.owasp.org/cheatsheets/Email_Validation_and_Verification_Cheat_Sheet.html. OWASP says not to activate accounts before verification completes.

- A pending record may hold the email, token hash, timestamps, and signup context. Some auth providers require an unverified account row. Keep it inactive and enforce the verified status on the server before granting access.
- Do not activate attacker-selected credentials when a mailbox owner confirms an unsolicited signup. Bind the confirmation to the intended signup, or let the verified owner set fresh credentials. Revoke earlier pending sessions and credentials when an owner claims the address.
- Before verification, give no credits, invites, trials, API keys, workspace resources, or session. Do not send product email other than the confirmation.
- Pending records must not squat an address. Uniqueness applies to verified emails only. A real owner can always start and finish verification, even when an attacker already created a pending row for that address. Postgres partial unique index example: `CREATE UNIQUE INDEX users_email_verified ON users (email_canonical) WHERE verified_at IS NOT NULL;`. Reference: https://www.postgresql.org/docs/current/indexes-partial.html
- Treat email as a weak factor. Verification proves mailbox access, not identity.
- Format validation does not prove mailbox access. Use a maintained library for syntax checks. Do not write a strict custom regex. A common practical maximum length is 254 characters.
- Disposable email domains: prefer risk scoring over blocking. OWASP advises risk-based controls instead of strict blocking. Block only with an explicit developer decision.

## Email canonicalization and aliases

Store the address exactly as the user typed it for delivery. Add a separate canonical key for comparison. OWASP advises this and warns against provider rules you do not fully control.

Rules:

- Lowercase the domain. Convert internationalized domains to punycode for comparison.
- Do not rewrite arbitrary local parts. Providers differ. A dot is meaningless on personal Gmail but changes the address on a Google Workspace or other custom domain. Google says dots do not matter for personal Gmail addresses and do change the address for work or school accounts. Source: https://support.google.com/mail/answer/7436150
- Build a provider-scoped abuse key only for domains whose rules you have confirmed. For example, for `gmail.com` only: lowercase, remove dots in the local part, drop a `+tag` suffix. Check Google's documentation before you add other domains such as `googlemail.com`.
- Use the abuse key for counting and flagging. Do not use it as identity. Never link a new signup to an existing account because the abuse keys match.
- Do not ban `+` aliases for everyone. Many real users use them to filter mail.

Illustrative config flags. Names are placeholders. Rename them to fit the project:

| Flag | Values | Default |
| --- | --- | --- |
| `SIGNUP_ABUSE_KEY` | `off`, `gmail-only` | `gmail-only`, log-only |
| `SIGNUP_ALIAS_POLICY` | `allow`, `flag`, `block-duplicate-verified` | `flag` |
| `SIGNUP_PLUS_ALIAS_POLICY` | `allow`, `flag` | `allow` |

- `flag` counts and logs. It does not block.
- `block-duplicate-verified` rejects a signup when a verified account already has the same abuse key. This is a developer decision. It affects real users who own several aliases of one mailbox. Resolve it only when you enable it. Use the generic response.

## Confirmation tokens

OWASP gives no numeric length or expiry. These are illustrative choices. Sources: https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html and the email cheat sheet above.

- Generate with a cryptographically secure random generator. Use at least 128 bits of randomness, encoded in URL-safe text.
- Store only a hash (SHA-256 is enough for random tokens). Link the token to one pending record.
- Make the token single-use and time-limited. Illustrative: 24 hours for signup, 48 to 72 hours for a waitlist.
- Consume it atomically: `UPDATE ... SET used_at = now() WHERE token_hash = $1 AND used_at IS NULL AND expires_at > now() RETURNING ...`. Zero rows means invalid.
- Build the link from a configured base URL. Never build it from the `Host` header.
- Send `Referrer-Policy: no-referrer` on the confirmation page. Use HTTPS only. Do not log tokens or full confirmation URLs.
- Mail scanners can open links and consume tokens. For stronger flows, make the link open a page with a confirm button that sends a POST request.
- Rate-limit confirmation attempts per IP.
- Issue replacement tokens only after resend limits pass. Avoid rotating valid tokens on every anonymous duplicate request; an attacker could invalidate the owner's link repeatedly.

## Waitlist

Keep three things separate:

1. **Mailbox confirmation.** The user clicked the link. This proves control of the mailbox. It does not create an account and it does not log anyone in.
2. **Authentication.** A later signup or login. A confirmed waitlist email grants no access by itself.
3. **Marketing or product consent.** A separate, recorded choice with its own text. Do not infer it from the confirmation click. Send no marketing to unconfirmed addresses.

Store status as `pending` or `confirmed`. Keep consent in its own fields. Apply the same checks as signup, including Turnstile, honeypot, and rate limits.

## Field validation and injection

Validation has two jobs. It keeps junk out of the data, and it gives users fast feedback. It does not stop injection.

Source: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html

- **The server decides.** Validate every field on the server. The frontend checks the same rules for fast feedback only. Share one rule definition between both when the stack allows it. Client-side checks are easy to bypass.
- **Names and short labels.** Normalize to Unicode NFC. Trim whitespace and limit length (illustrative: 1 to 100 characters). Reject control characters and bidirectional override characters. Allow broad Unicode and punctuation. Some writing systems use joiners; do not ban all zero-width characters. Accept a single name. Do not require two parts or ASCII.
- **URL-like, HTML-like, and code-like values.** In name and short label fields, reject values that contain a URL scheme (`://`), `www.`, angle brackets, template delimiters (`{{`, `${`), or backticks. Real names do not contain these. This rule fights spam and broken emails. It is not injection defense. Return a field error. Do not strip characters silently.
- **Free text** (company, message). Allow broad Unicode and punctuation. Limit length. Limit the number of links (illustrative: 0 to 2).
- **Email headers.** Reject CR and LF in any value that reaches an email header.
- **Injection defense happens at the sink:**
  - Use parameterized queries for SQL. Do not escape by hand. Source: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
  - Encode output for its context (HTML body, attribute, JavaScript, URL). This includes names in email templates and admin screens. Denylists and WAFs are unreliable. Source: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
  - Store the raw validated value. Do not store pre-encoded text.
- Do not ban apostrophes, hyphens, or SQL words. A ban rejects real people such as O'Brien and does not make a query safe.
- Log rejected input escaped and truncated. Do not log full request bodies.

## Generic responses and enumeration

Source: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html

- Return one message for new, existing, pending, and rate-limited-by-recipient cases. Example: "Check your email to continue."
- Keep the status code and body shape the same. Keep timing close. Queue the email send so the response does not wait for it.
- If an account already exists, send that address an email that says so, with a sign-in link. Do not say it on the page.
- Rate limits keyed to the client IP may return 429. Do not return a different response because of whether an address exists.

## Rate limits and 429

- Apply limits on the server with atomic counters. A read followed by a write lets parallel requests slip through. Use one atomic operation: Redis `INCR` plus `EXPIRE` in a Lua script or `MULTI`/`EXEC` (https://redis.io/docs/latest/commands/incr/), or a Postgres `INSERT ... ON CONFLICT DO UPDATE SET count = count + 1 RETURNING count`. In-memory counters per instance fail on serverless and multi-instance hosts.
- Return `429 Too Many Requests` with a `Retry-After` header in seconds. Do not cache it. Sources: https://datatracker.ietf.org/doc/html/rfc6585#section-4 and https://www.rfc-editor.org/rfc/rfc9110#section-10.2.3
- Limit per client IP, per recipient address, and globally. The numbers are in [tuning-and-operations.md](tuning-and-operations.md).
- A per-recipient limit must stop attackers from using your form to flood a victim's inbox. Count every request for that address, whether or not an account exists.
- When a per-recipient budget runs out, return the generic success response and send nothing. This avoids revealing the address state.

## Cleanup

- Delete expired pending records and their tokens on a schedule (illustrative: daily job, delete pending rows older than 7 days).
- Keep only the data needed for abuse counting after deletion, such as hashed counters with short expiry.
- Race-safe deduplication: rely on the database unique index, not on a check followed by an insert. Catch the unique violation and return the generic response.
