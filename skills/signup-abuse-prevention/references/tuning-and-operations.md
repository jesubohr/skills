# Tuning and Operations

Sources reviewed on 2026-10-29. Every number in this file is an illustrative starting point from this skill. None is a standard. Tune each number with your own traffic. Start in log-only mode when a limit is new.

## Contents

- Starter profiles
- What the limits count
- Spend caps
- Provider outages
- Monitoring and false positives
- Rollback
- Verification checklist

## Starter profiles

Pick the smallest profile that fits. Provider defaults already limit some routes. For example, Supabase Auth limits sign-up routes to 30 requests per 5 minutes per IP by default. Source: https://supabase.com/docs/guides/auth/rate-limits. Set your own limits below the provider's limits when you want tighter control.

| Control | Baseline (small, quiet) | Elevated (measured abuse) | Under attack (temporary) |
| --- | --- | --- | --- |
| Signup attempts per IPv4, per hour | 10 | 5 | 2 |
| Signup attempts per IPv6 `/64`, per hour | 10 | 5 | 2 |
| Signup attempts per IPv6 `/48`, per hour | 50 | 25 | 10 |
| Confirmation emails per recipient, per hour | 3 | 2 | 1 |
| Confirmation emails per recipient, per day | 6 | 4 | 3 |
| Resend cooldown per recipient | 60 seconds | 120 seconds | 300 seconds |
| Confirmation attempts per IP, per hour | 20 | 10 | 5 |
| Verified accounts per IP, per 24 hours | no cap | 2, step up when exceeded | 1 to 2, review when exceeded |
| Phone OTP | off | step-up for risky signups only | step-up for risky signups only, restricted countries |
| IP intelligence | off | log-only score | score plus selective step-up |
| Global confirmation emails per day | 10 x normal daily volume, minimum 200 | 5 x normal | 3 x normal |

Notes:

- "Normal daily volume" is your measured average over recent weeks. With no data, start from the minimum value and adjust.
- An IP limit slows bulk attempts. It does not identify a person. Do not store IP counters as a permanent record.
- Per-recipient limits apply whether or not an account exists for the address.
- Waitlist forms can use the baseline column with the same numbers.
- Shared networks affect attempt limits as well as account caps. For an event with 150 attendees on one IP, 10 attempts per hour would block most attendees. Set a temporary event-specific attempt budget with headroom, or use a signed event path. Keep per-recipient limits and bot checks. Expire the exception after the event.

## What the limits count

- Count attempts, not only successes. Failed Turnstile checks, honeypot hits, and invalid tokens all count toward the IP limit.
- Key counters by a keyed HMAC where useful, for example of the normalized recipient address. A plain hash of an email or IP is easy to guess and is not anonymous. Expire counters and retain only necessary data.
- Use one atomic increment per check. See "Rate limits and 429" in [baseline-controls.md](baseline-controls.md).
- On a limit hit, return `429` with `Retry-After`. For a per-recipient limit, return the generic success response and send nothing.

## Spend caps

Email and SMS sends cost money, and attackers can use them to cost you money.

- Set a global daily cap for emails and a separate, lower one for SMS or WhatsApp. Count sends, not requests.
- Alert at 50% and 80% of each cap (illustrative).
- At the cap, stop new sends. Show a neutral message such as "We cannot send email right now. Try again later." For a waitlist, store the entry as pending and send later when the cap resets.
- Set a spend limit in the provider's console when the provider offers one. Provider limits are a second layer. They do not replace your own cap.
- Review the caps after each traffic change, such as a launch or a press mention. Raise them on purpose and write down the reason.

## Provider outages

Decide each policy once. Write it in the code as a named setting.

- **Turnstile unavailable** (timeouts or `internal-error` after retries). Choose one:
  - Fail closed: reject the signup with a retry message. This is the safest and the default.
  - Fail open with lower limits: accept the signup into a pending state with stricter IP and recipient limits and mandatory email verification. Only the developer chooses this. Log every fail-open event.
  - Do not fail open for the phone-OTP route or for any send that costs money.
- **Email provider unavailable.** Queue the confirmation and retry with backoff. Do not drop the signup. Do not skip verification. Keep the spend cap. Let queued jobs expire. Do not send a burst of old messages when the provider returns. Do not switch to another provider or domain to avoid enforcement.
- **SMS provider unavailable.** Show a retry message. Do not bypass phone verification when the signup requires it.
- **IP intelligence unavailable.** Skip the signal. Do not block because a lookup failed.
- **Rate-limit store unavailable.** For a baseline route, fail closed on sends that cost money. Choose fail-open only for forms that send no email. Log it.
- Tell the user clearly when you picked a policy, and which one.

## Monitoring and false positives

Track these numbers per day. Compare them to a normal week.

- Signup attempts, by route.
- Turnstile failure rate and error codes.
- Honeypot hits.
- Rate-limit hits, by rule name.
- Confirmation conversion: confirmed divided by sent. A sharp drop can mean abuse or a delivery problem.
- Email bounce, spam complaint, and delay counts and rates, per stream, with time window and denominator. Rising rates hurt your sender reputation. Alert on counts at low volume. See [email-reputation.md](email-reputation.md).
- Verified and activated accounts, apart from pending rows and sends.
- Outbox queue depth, expired jobs, and suppressed-recipient skips.
- Flags and blocks by rule name, including log-only results.
- Phone OTP conversion (verified divided by sent), by country.
- Spend against each cap.
- Support contacts from users who say they could not sign up.

False-positive practice:

- Record the rule name with every block, flag, or step-up. Use a reason code. Do not store extra personal data to do this.
- Run a new rule in log-only mode first. Review a sample of flagged signups. Enable the action when the sample looks right.
- Give blocked users a way to reach support. Offer a manual approval path for real people.
- Review shared-network cases first: offices, schools, carriers, events.
- Remove a rule when it blocks real users more than it stops abuse.

## Rollback

- Give every control its own setting that turns it off or into log-only without a deploy when possible. Examples: `SIGNUP_ALIAS_POLICY=allow`, `SIGNUP_IP_CAP=off`.
- Write down the previous value before each change. Revert one setting at a time.
- Under-attack settings are temporary. Set a review date and move back down a tier when the attack ends.
- Keep the baseline on during a rollback of a tier control.

## Verification checklist

Run these against a staging environment or local test setup. Use test keys. Do not call private production systems or paid services.

- Each route configured to require Turnstile rejects a missing token, a wrong configured `action` or `hostname`, and a reused token. Test provider-managed verification through its supported controls.
- The origin API and the hosted auth endpoints reject direct calls that skip the frontend.
- A request with a filled honeypot creates no record and sends no email, and returns the normal response.
- A forged `X-Forwarded-For` value does not change the counted IP.
- Parallel requests (for example 20 at once) do not exceed an IP or recipient limit.
- The same email returns the same status, body shape, and nearly the same timing for new, pending, and existing addresses.
- Email bombing from rotating IPs, a suppressed recipient with queued sends, duplicate webhook callbacks and retries, malicious name content in mail, and analytics that exclude pending rows: run the tests in [email-reputation.md](email-reputation.md).
- A pending record gets no credits, invites, session, or resources. A verified user does.
- An expired, used, or altered token fails. Two parallel confirmations succeed once.
- A second pending signup for one address does not block the real owner from verifying.
- Names such as `Zoë`, `O'Brien`, `李雷`, and a single name pass. A name with `<script>`, a URL, or a newline fails, and stored values render encoded in the admin and email views.
- A `+` alias passes by default. The abuse-key flag appears only when the setting is on.
- The cleanup job removes expired pending rows.
- Turning each control off through its setting works.
