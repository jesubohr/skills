# Risk Tier Controls

Read this file when the baseline is not enough and the user has real abuse to stop. Do not enable these controls by default.

Sources reviewed on 2026-10-29. Prices and provider behavior are volatile. Numbers marked "illustrative" are examples from this skill, not standards.

## Contents

- Tiers
- Account cap per IP
- IPv6 strategy
- Phone OTP: SMS and WhatsApp
- Country restrictions
- IP intelligence
- Name and placeholder checks
- Disposable email

## Tiers

| Tier | When to use it | Controls |
| --- | --- | --- |
| 0. Baseline | Always | See [baseline-controls.md](baseline-controls.md) |
| 1. Observe | Abuse is suspected but not measured | Log-only flags and risk score: abuse key, IP type, disposable domain, placeholder name, fast completion |
| 2. Step up | Abuse is measured and ongoing | Lower limits for risky traffic, an account cap per IP, stricter Turnstile settings, phone OTP for risky signups only |
| 3. Under attack | Active, costly attack | Temporary hard blocks, country limits, a manual review queue. Remove them when the attack ends |

Move up one tier at a time. Move down when the attack ends. Ask the developer at the point where a control restricts real users or spends money.

## Account cap per IP

An optional cap of 1 to 2 accounts per IP stops cheap bulk signups. It also hurts real users who share an IP. These users include offices, schools, universities, cafes, mobile carriers that share one address across many phones, and people at an in-person event.

- Count verified accounts, or confirmed signups, per client IP in a rolling window. Illustrative: 2 per 24 hours. Do not use a lifetime count. An IP address is not a permanent identity. Addresses change owners.
- Count pending attempts separately, with the rate limits in [tuning-and-operations.md](tuning-and-operations.md).
- Prefer a soft action when the cap is hit: ask for phone OTP, or place the signup in a review queue. Use a hard block only after the developer decides.
- The developer decides before the cap is enabled:
  - Which networks are shared? Add known office or school IPs to an allowlist.
  - Is there an in-person event or promotion? If yes, raise the cap for that IP and window, or issue event codes that bypass the cap. Remove the exception after the event.
  - What does a blocked person see? Show a message and a support contact. Do not leave a dead end.
- Use IPv6 buckets from the next section.

## IPv6 strategy

One home or office often receives a whole IPv6 block. An attacker can rotate addresses inside it. A per-address limit is useless there.

This is common practice, not a standard. Check it against your own traffic:

- Count IPv4 per full address (`/32`).
- Count IPv6 per `/64` as the main bucket. Add a looser second bucket per `/48` or `/56`. Illustrative: allow the wider bucket 5 to 10 times the `/64` limit.
- Store the bucket key, not the raw address, when you only need counting.
- Behind Cloudflare with Pseudo IPv4 enabled, the client IP header can hold a pseudo IPv4 address. The real IPv6 address is in `CF-Connecting-IPv6`. Source: https://developers.cloudflare.com/fundamentals/reference/http-headers/

## Phone OTP: SMS and WhatsApp

A phone check raises the cost of fake accounts. It also costs money on every send, adds friction, and attracts SMS pumping. Use it as a step-up for risky signups. Do not add it to every signup.

### Cost

Cost per verified signup is: verification fee + message fee x messages sent. Count resends and failed attempts, because the message fee still applies.

Twilio Verify, from the pricing pages dated October 2026. Source: https://www.twilio.com/en-us/verify/pricing and https://www.twilio.com/en-us/sms/pricing/us

- $0.05 per successful verification, plus channel fees.
- US SMS: $0.0083 per message. The SMS pricing page also lists per-message carrier fees for US carriers (for example $0.0035 to $0.005). Check whether they apply to your setup.
- Failed, expired, or incorrect verifications do not incur the $0.05 fee. The SMS or WhatsApp delivery can still be billed.
- International SMS, voice, email, and silent network auth rates are on linked pages. They vary by country and carrier.
- WhatsApp: the Verify page lists two different figures for the authentication template ($0.0034 and $0.0147). Verify before you rely on either.
- Illustrative math, US only, before carrier fees: 1,000 verified signups with 1.3 SMS each is about $50 + 1,300 x $0.0083 = $60.79.

WhatsApp Business Platform, per Meta (page content reflects rates effective October 1, 2026): Meta charges per delivered business message. The rate depends on the recipient's country and the message category. Rates change on the first day of a quarter. Rate cards are in linked CSV and PDF files. Source: https://developers.facebook.com/docs/whatsapp/pricing/

Do not copy prices into code or docs without a date. Read the current pages first.

### Toll fraud and SMS pumping

Attackers abuse phone-number fields to trigger paid messages to numbers they profit from. Twilio lists these controls. Source: https://www.twilio.com/docs/verify/preventing-toll-fraud

- Turn on the provider's fraud tool (Twilio SMS Fraud Guard).
- Disable channels you do not use, such as voice.
- Allow only destination countries you serve. This is the highest-value control. Apply tighter limits to countries with little real traffic.
- Rate-limit per phone number, per number prefix, per IP, and per user. Use growing delays between resends.
- Run bot detection and require email confirmation before any phone step.
- Use carrier lookup to allow mobile line types only, when that fits your users.
- Watch the OTP conversion rate (verified codes divided by sent codes). Alert on a drop, especially in unexpected countries.
- Set a hard daily spend cap and an alert. See [tuning-and-operations.md](tuning-and-operations.md).
- Consider selective Tor and proxy blocks on the phone-OTP route after the developer resolves that policy. Treat hosting and VPN flags as risk signals, not proof.

Twilio states that no provider-side solution can guarantee full protection. Rate limits slow attackers. They may not stop them.

### OTP design

- Prefer provider-managed codes. For a custom flow, use a cryptographically random 6-digit code (illustrative), about 10 minutes of validity, and at most 5 attempts per challenge across resends. Bind it to the phone, pending signup, and purpose. Consume it atomically once. Store a keyed HMAC with a server-held key; a plain hash of a short code is easy to exhaust offline.
- Use `autocomplete="one-time-code"` on the input.
- Phone numbers are personal data. Collect one only when you use it. Do not require it in the baseline. Tell users why you ask.

## Country restrictions

Use these only when the service is genuinely scoped to some countries, for example by license, shipping area, language, or support coverage.

- IP country is a good but imperfect signal. MaxMind reports 99.8% country-level accuracy for its own data. It reports much lower accuracy for states and cities. VPNs, mobile networks, and hosted traffic show the server or carrier location. Source: https://support.maxmind.com/knowledge-base/articles/maxmind-geolocation-accuracy
- Separate two rules:
  - **SMS destination countries.** Restrict these early. They cut pumping cost and rarely hurt real users.
  - **Signup source countries.** Prefer step-up (extra check, lower limits) to a hard block. Hard-block only when the product scope requires it.
- If the country is unknown, do not block. Apply the normal flow.
- Legal restrictions such as sanctions or export rules need legal advice. This skill does not decide them.
- Let blocked users reach support.

## IP intelligence

Use IP type as a risk input. Do not use it as a verdict.

### DIY lists (free, partial)

- Tor exit nodes: https://check.torproject.org/torbulkexitlist (plain text, one IPv4 per line, about 1,500 to 2,000 entries when checked).
- AWS ranges: https://ip-ranges.amazonaws.com/ip-ranges.json. AWS publishes ranges for commonly used services. It does not publish all ranges and it excludes BYOIP ranges. Download over verified TLS and keep the publication time. Source: https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html
- Google Cloud ranges: https://www.gstatic.com/ipranges/cloud.json. Source: https://docs.cloud.google.com/compute/docs/faq
- Other hosting providers publish their own lists. Find them in their documentation.
- Refresh on a schedule (illustrative: daily). Cache the last good copy. If the download fails, keep using the cache.
- Limits: DIY lists miss many VPNs, small hosts, and residential proxies. They also include legitimate users, such as employees whose traffic exits through a cloud network.
- Do not treat your own CDN or proxy ranges as client IPs.

### Paid data

MaxMind GeoIP Anonymous IP, GeoIP Insights, and minFraud return anonymizer flags (VPN, hosting provider, public proxy, residential proxy, Tor exit node). Advanced products add a confidence score from 1 to 99, `provider_name`, and `network_last_seen`. Source: https://support.maxmind.com/knowledge-base/articles/anonymizer-and-proxy-data-maxmind

MaxMind's guidance for residential-proxy confidence scores follows. VPN scores have different meanings; do not apply these ranges to every product:

- 80 to 99: suits aggressive blocking if your risk tolerance allows.
- About 50 to 75: add checks to reduce false positives.
- 1 to 45: a weak prior.
- 1: do not block. These IPs are often shared infrastructure such as carrier-grade NAT.
- Use `provider_name` to avoid blocking corporate VPNs. Use `network_last_seen` to skip stale entries.
- Apple iCloud Private Relay and similar services are not flagged as anonymizers.

Pricing differs by vendor and changes often. Read the vendor's current pricing page. Do not subscribe or call a paid API unless the user asks.

### Paid or DIY?

| Need | Choice |
| --- | --- |
| Low volume, Tier 0 or 1 | Skip IP intelligence, or use the free Tor and cloud lists as a score input |
| Repeated abuse from hosting, VPN, or proxy IPs | A paid database or API. It saves maintenance and gives confidence scores |
| Phone OTP route | Consider selective Tor and public-proxy blocks after a developer decision. Add paid data if pumping appears |

### Selective action

| Signal | Suggested action |
| --- | --- |
| Tor exit or public proxy | Step up (extra check, lower limits). Any block, including on the phone-OTP route, is a developer decision |
| Hosting provider | Add to the risk score. Do not block alone |
| Commercial VPN | Add to the risk score. VPNs have legitimate uses. Twilio calls VPN a signal, not proof |
| Residential proxy, high confidence | Step up |
| Carrier-grade NAT or mobile network | Never block on IP type |

Start in log-only mode. Review the false positives. Then enable actions one signal at a time.

## Name and placeholder checks

Use placeholder checks as a soft score. Never reject on them alone.

- Compare the whole normalized field to a short configured list of placeholders, for example `test`, `asdf`, `qwerty`, `n/a`. Match the whole value only, case-insensitive. Do not match substrings. `Contest` and `Tester` must pass.
- Do not ship a name dictionary. Do not block ordinary words as names. Real people have unusual names and names that are also common words.
- Do not check for ASCII-only text or for two-part names.
- Random-string detection is unreliable across languages. Skip it unless data shows a need.
- A pattern across many signups is stronger than one name. Examples are the same name with many emails, or many sequential emails from one network in a short time.
- Illustrative flag: `SIGNUP_PLACEHOLDER_NAMES` (list, empty by default, action `flag`).

## Disposable email

- Use a maintained disposable-domain list as a risk score. OWASP advises risk-based controls over strict blocking.
- Block disposable domains only with an explicit developer decision, for example when free credits are the target.
- Update the list on a schedule. Allow an override for false positives.
