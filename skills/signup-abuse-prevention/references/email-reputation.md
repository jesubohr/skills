# Email Reputation

Sources reviewed on 2026-10-29. Each section marks provider rules (from linked documents) and engineering examples (illustrative choices from this skill). No control here guarantees that mail avoids spam folders or flags. Provider rules and limits change, so check the linked pages before you rely on them.

## Contents

- Why verification alone is not enough
- Gate every anonymous send
- Budgets and velocity caps
- Reservations, outbox, and workers
- Bounce and complaint feedback
- Suppression and recovery
- Streams and domains
- Authentication DNS
- Message content
- Metrics that mean real use
- Monitoring and pause
- Tests

## Why verification alone is not enough

Email verification proves mailbox control after the message arrives. The first verification message still goes to an address the requester chose. An attacker can use your form to send that message to many victims. Victims then mark it as spam, and your sender domain pays for it.

Treat every anonymous request that can cause an email as an outbound-mail risk, not only an account risk.

## Gate every anonymous send

Apply the same gate to each route where an unauthenticated person can cause a message to a chosen address:

- signup confirmation and the "account already exists" notice
- waitlist confirmation
- resend of any message
- magic links and sign-in codes
- password reset
- invitations that an anonymous or unverified person can trigger

Before the first message, the gate must pass: bot validation (see Turnstile in [baseline-controls.md](baseline-controls.md)), the per-IP budget, the per-recipient budget, and the global and recipient-domain caps below. Authenticated, verified users who invite others need their own per-user budget. Do not skip the gate on a route because its form looks harmless.

## Budgets and velocity caps

An engineering example. Numbers for the first three rows are in [tuning-and-operations.md](tuning-and-operations.md).

- **Per IP** (and IPv6 `/64`): limits the number of sends one client can cause.
- **Per recipient** with a cooldown: limits sends to one mailbox, across all routes together. One shared counter per recipient is simpler and safer than one counter per route.
- **Global** daily and hourly cap on anonymous-origin sends.
- **Per recipient domain** velocity cap. A sudden rise of sends to one small domain, such as a company or school domain, is a sign of bombing a group of victims or of probing for spam traps. Alert and slow down. Do not block the domain, because many real users share it.
- **Distributed bombing.** Attackers rotate IPs, so per-IP limits alone fail. The per-recipient budget, the recipient-domain cap, and the global cap are the backstops. Bot validation raises the cost of each IP.
- **Alias abuse key** (optional). Use the provider-scoped key described in [baseline-controls.md](baseline-controls.md) for the recipient budget, so `a.b+x@gmail.com` and `ab@gmail.com` share one budget. Keep delivery on the original address. Use the key only for counting. Do not link accounts or identities with it.

When a budget is exhausted, return the same generic response and send nothing. See "Generic responses and enumeration" in [baseline-controls.md](baseline-controls.md).

## Reservations, outbox, and workers

Engineering example. The aim is that no path sends more than the budget allows, even under retries and parallel requests.

- **Reserve atomically.** In one atomic operation, check the budgets and record a send reservation. Do not read a counter, then write it later.
- **Outbox with deduplication.** Write one outbox row per intended message, with a unique key such as recipient, message type, and window. A retried request or a duplicate callback then cannot create a second message.
- **Bounded retries.** Retry only transient failures (timeouts, provider throttling, soft bounces) with backoff and a fixed attempt limit. Do not retry permanent failures.
- A timeout can occur after the provider accepts a message. Use provider idempotency when available, or reconcile the delivery state before retrying. An outbox alone cannot guarantee exactly one external send. Do not charge a reserved send twice when the worker rechecks its budget.
- **Recheck at dispatch.** The worker must recheck suppression, budgets, and caps immediately before it sends. This includes queued and replayed jobs. A job that was valid when it was queued can be invalid now.
- **Expire queued jobs.** Give each confirmation or sign-in message a short life (illustrative: shorter than the token life). After an outage or an attack, expired jobs are dropped. Do not send a catch-up burst of old messages.

## Bounce and complaint feedback

Use the provider's delivery feedback. Amazon SES is one example. Provider facts below come from the AWS pages for SES. Other providers differ.

Sources: https://docs.aws.amazon.com/ses/latest/dg/notification-contents.html and https://docs.aws.amazon.com/ses/latest/dg/send-email-concepts-deliverability.html

- **Authenticate the webhook.** Accept bounce and complaint callbacks only with a verified provider signature or a shared secret that the provider supports. Reject all other requests. Use the provider's own verification steps. Do not trust the body alone.
- **Map the recipient on the server.** Match the event to your own stored message through the provider message ID and the recipient. Do not trust an address that arrives only in the body to choose which user to suppress.
- **Deduplicate events.** Providers can deliver a notification more than once, and SES documents at-least-once delivery. Store the event ID (for SES, `feedbackId`) and ignore repeats. Handle one event that lists several recipients, and unknown new fields.
- **Hard bounce.** A permanent failure (SES `bounceType: Permanent`). Suppress the address at once. AWS advises against repeated attempts to hard-bounce addresses.
- **Complaint.** Suppress the address at once. AWS advises against repeated attempts to addresses that complain. A complaint report can list recipients that only might have complained, so keep the recipient-to-message mapping.
- **Soft bounce.** A transient failure (SES `bounceType: Transient`, such as a full mailbox). Do not suppress on the first one. Retry with a bound, and suppress after repeated failures. Check the provider's own retry behavior so that you do not retry on top of it.
- **Undetermined.** Treat it as a transient failure with a low retry limit, and review the counts.

## Suppression and recovery

Source: https://docs.aws.amazon.com/ses/latest/dg/sending-email-suppression-list.html

- Keep suppression in your app as well as in the provider. Provider lists apply to one account or region and depend on settings. Enable both bounce and complaint suppression. SES automatically adds bounce entries for hard bounces, not soft bounces; it can also add complaint entries. SES does not receive Gmail complaint data. Do not treat missing events as proof of no complaints.
- Check the app suppression before every send, in the gate and again at dispatch.
- **Signup, resend, and sign-in requests must never clear suppression.** An anonymous request must not remove a bounce or complaint record, and must not reset budgets for the address.
- **Recovery is a reviewed path.** A real owner whose address was suppressed by mistake needs a way back: a support request that a person reviews, an address-ownership check on another channel, then a manual removal in the app and the provider. A permanent lockout from one accidental complaint is a product failure. Record who removed the suppression and why.
- Do not rotate to another provider, sender domain, or IP pool to avoid enforcement or suppression. Fix the cause: reduce anonymous sends, add the gate, and clean the list. Rotation moves the damage to a second reputation.

## Streams and domains

Engineering example, with provider context.

- Separate critical transaction mail (sign-in, reset, receipts) from marketing and waitlist mail. Use separate provider streams, configuration sets, or sender subdomains, with separate From addresses. A problem in the waitlist stream then does not stop sign-in mail.
- Caveat: reputation can still be shared through the parent domain or the sending IP pool. Separation lowers risk. It does not remove it.
- Do not buy a dedicated IP by default for low volume. A dedicated IP gives the sender the whole reputation task, and low, irregular volume gives providers little history to judge. Revisit it when volume is high, steady, and measured.
- Gmail asks bulk senders not to mix content types, such as promotions inside receipts. Source: https://support.google.com/mail/answer/81126?hl=en

## Authentication DNS

Provider rules: Gmail requires SPF or DKIM for all senders, and SPF, DKIM, and DMARC for bulk senders (more than 5,000 messages per day to Gmail). The From domain must align with the SPF domain or the DKIM domain. Sources: https://support.google.com/mail/answer/81126?hl=en and https://support.google.com/mail/answer/14229414

- Use the setup steps from your email provider for DKIM and SPF. Check DMARC alignment of the visible From domain.
- Start DMARC with a monitoring policy (`p=none`) and read the reports before you move to a stricter policy. This is an engineering example.
- **Review existing DNS before you change it.** List the current SPF, DKIM, and DMARC records first. A domain can have only one SPF record, and an overwrite can break other senders such as the company mail system. Merge new entries. Do not replace records blindly. Ask the developer to apply DNS changes unless they gave you access and approval.
- Verify with the provider's check or a test message, not with a guess.

## Message content

- Confirmation emails use a short, recognizable, fixed subject, From name, and sending domain. Example: "Confirm your email for Acme".
- Do not reflect user-submitted values into the subject or body: not the name, message, company, or any URL. Escaping does not fix this. An attacker can still use your domain to deliver scam text to a victim ("relay"). Fixed text is the safe choice. See field validation in [baseline-controls.md](baseline-controls.md) for output encoding in other views.
- Send no promotions, product news, or marketing before the recipient has consented. See "Waitlist" in [baseline-controls.md](baseline-controls.md).
- Marketing messages to Gmail need one-click unsubscribe when the bulk-sender rules apply. Source: https://support.google.com/mail/answer/14229414

## Metrics that mean real use

Pending rows, raw form submissions, and email sends are not usage. Count these separately:

- **Verified accounts.** The mailbox owner completed verification. A verified email alone is not proof of genuine use, because attackers can own mailboxes.
- **Activated accounts.** The account did a meaningful, authorized action in the product. Define the action once.
- **Activity metrics.** Count only actions by verified, authorized accounts. Exclude pending rows, bots flagged by the gate, honeypot hits, and sends.

Report conversion as verified divided by pending, and activated divided by verified. A rise in pending rows without a rise in activation is an abuse signal, and it also inflates statistics and resource use.

## Monitoring and pause

Source for Gmail thresholds: https://support.google.com/mail/answer/81126?hl=en. Gmail says to keep the Postmaster Tools spam rate below 0.1% and to avoid 0.3% or higher. These are Gmail targets, not guarantees of inbox placement. The reviewed pages do not describe data for low-volume senders. Expect missing or noisy data at low volume.

Engineering examples:

- Track bounce, complaint, and delay (defer) counts with the time window, per stream. Show the numerator and denominator. At low volume, one event changes a rate a lot. Alert on counts as well as rates, for example more than a small fixed number of complaints in one hour.
- Use the denominator that the provider uses. SES, for example, excludes messages to suppressed addresses from its reputation rates and counts only hard bounces in the bounce rate. Gmail Postmaster reports its own user-reported spam rate. Do not compare numbers from different providers as if they were the same.
- Alert early, before a provider threshold. Use your own thresholds, based on measured history.
- **Pause automatically.** When the anonymous-origin stream passes an alert threshold, pause that stream (waitlist, signup confirmations from unknown addresses) and queue or drop by expiry. Keep mail to known, verified users flowing when it is safe: sign-in, reset for verified accounts, and receipts. Resume by hand after a review.
- Record the reason code with every pause.

## Tests

Run these in staging with a provider simulator or a fake sender. Do not send to real third-party addresses.

- Bombing from rotating IPs: many requests for one recipient, and many for one recipient domain, from different IPs. The sends stop at the recipient and domain budgets.
- Parallel requests for one recipient create one reservation.
- A suppressed recipient with a queued or replayed job sends nothing at dispatch.
- A signup, resend, or sign-in request for a suppressed address does not clear the suppression.
- A duplicate webhook callback, a retried request, and a repeated job each cause one effect.
- A callback with a bad signature, or with a recipient that does not match the stored message, changes nothing.
- A soft bounce retries within the limit and then stops. A hard bounce and a complaint suppress at once.
- A name or message such as `Pay now at http://example.test` never appears in a subject or body of any outgoing email.
- Expired queued jobs are dropped after a simulated outage. No burst follows.
- The anonymous stream pauses at the alert threshold, and sign-in mail for a verified user still sends.
- Analytics and activity metrics exclude pending rows, honeypot hits, and sends.
