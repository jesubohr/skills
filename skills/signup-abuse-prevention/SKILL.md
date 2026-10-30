---
name: signup-abuse-prevention
description: Reduce bot abuse on signup, registration, and waitlist forms of small SaaS products. Stops fake accounts, inflated usage statistics, drained resources, and damage to the sender email and domain reputation. Recommends and implements a small set of controls, such as Cloudflare Turnstile, a honeypot field, email verification, rate limits, email alias handling, field validation, safe email sending with bounce and complaint handling, and optional phone OTP or IP intelligence. Use when the user reports fake signups, spam waitlist entries, email or SMS bombing, SMS pumping, free-credit abuse, inflated user counts, or spam flags and bounces on signup mail, or asks to harden a signup flow. Not for login brute force, payment fraud, or full fraud platforms.
---

# Signup Abuse Prevention

Reduce signup and waitlist abuse with the fewest controls that work. Keep signup available to real users. Protect three things: account and usage data, product resources, and email sender reputation. These controls reduce risk; they cannot guarantee that every bot is blocked or every email reaches the inbox.

Controls fall into a small baseline and optional risk tiers. Do not enable every control by default. Each added control adds false positives, cost, or both.

## Scope and decisions

- Work inside the user's request and their existing authorization. Ordinary code changes need no extra approval from this skill.
- Resolve project decisions before enabling alias blocks, an account cap per IP, country or IP-type blocks, or paid phone verification. Use existing user authorization when it covers the decision. Otherwise ask only for the missing decision.
- Until the developer decides, leave that control off or in log-only mode. Continue the independent baseline work.
- Do not call private production systems or paid services unless the user asks. Prefer test keys and local tests.
- Do not use invasive fingerprinting by default.

## Workflow

1. **Inspect the flow.** List every route that creates an account, joins a waitlist, or sends an email or SMS. Include the origin API, hosted auth endpoints, social login, magic links, invites, resend, and password reset. A control on the form alone does not protect these routes.
2. **Choose a profile.** Read [references/tuning-and-operations.md](references/tuning-and-operations.md) and pick the baseline. Add a tier only for a real, observed problem.
3. **Implement the controls.** Read [references/baseline-controls.md](references/baseline-controls.md) for the baseline. Read [references/risk-tier-controls.md](references/risk-tier-controls.md) only when you add a tier control. Read [references/email-reputation.md](references/email-reputation.md) when any anonymous request can cause an email to a chosen address, or when the user reports spam flags, bounces, or complaints.
4. **Verify.** Test each route directly, without the frontend. Confirm the control runs on the server.

## Baseline

- Turnstile with server-side verification, plus a CSS-hidden honeypot field.
- Email verification before any activation. A pending record is allowed. No credits, invites, or resources before the email is verified.
- Server-side field validation that matches the frontend rules. Parameterized queries and contextual output encoding stop injection. String bans do not.
- Atomic rate limits per IP and per email recipient, with a global daily email cap.
- Generic responses that do not reveal whether an email exists.
- Every anonymous email send passes the same gate first. Email verification alone does not stop bombing, because the first message goes to any address. See the email reputation reference.
- Usage statistics count verified, activated accounts. Pending rows, raw submissions, and sends are not usage.
- Monitoring and a kill switch for each control.

## Rules that prevent harm

- Do not block `+` aliases or ordinary words or names with a blanket rule.
- Keep the original email address for delivery. Use a provider-scoped canonical key only for abuse counting. Do not rewrite arbitrary local parts. Do not link an alias to an existing account automatically.
- Do not use name dictionaries, ASCII-only rules, or required two-part names.
- An IP address is an attempt signal, not a permanent identity. Shared networks, mobile carriers, and offices share IPs.
- A single weak signal, such as a fast form completion, never blocks a user alone.
- A waitlist confirmation proves control of a mailbox. It is not authentication and it is not marketing consent. Keep the three separate.
- Facts about limits, prices, and provider behavior change. The numbers in the references are illustrative examples or dated provider facts. They are not standards.

## References

- [references/baseline-controls.md](references/baseline-controls.md): how to build each baseline control, alternate routes, proxy IP trust, and the email and token design.
- [references/risk-tier-controls.md](references/risk-tier-controls.md): per-IP account caps, IPv6, phone OTP and costs, country limits, IP intelligence, and name heuristics.
- [references/email-reputation.md](references/email-reputation.md): send gates, budgets, outbox and workers, bounce and complaint webhooks, suppression, streams, DNS authentication, safe message content, real-use metrics, and tests.
- [references/tuning-and-operations.md](references/tuning-and-operations.md): starter profiles with numbers, spend caps, outage handling, monitoring, and rollback.
