# United States

Reference date: October 2026. US law changes often, mostly at the state level and through court decisions. Confirm each item with a web search before you rely on it. Items marked `(verify)` had conflicting or secondary sources.

## Contents

1. Big picture
2. Privacy policy (CalOPPA)
3. State comprehensive privacy laws
4. Opt-out rights, "Do Not Sell or Share", and GPC
5. Cookies, pixels, and litigation risk
6. Minors (COPPA and state laws)
7. Terms enforceability (clickwrap)
8. Refunds
9. Subscriptions and auto-renewal
10. Marketing email and SMS
11. Health data
12. Data breach notification
13. Accessibility
14. Copyright (DMCA)
15. Checklist

## 1. Big picture

- There is no single federal privacy law. Federal rules are sector-based (COPPA, HIPAA, GLBA, FTC Act). The FTC enforces "unfair or deceptive practices": if the privacy policy says one thing and the product does another, that is a deception case.
- About 20 states have comprehensive privacy laws. Most apply only above size thresholds, but California's privacy-policy law (CalOPPA) applies to every site with California users.
- Private lawsuits are the main risk for small startups: wiretap claims for pixels and session replay (CIPA), video tracking (VPPA), accessibility (ADA), and SMS (TCPA).

## 2. Privacy policy (CalOPPA)

California Business and Professions Code 22575. Applies to any operator of a site or app that collects personal data from California residents. Size does not matter. Because of this, every US-facing product needs a privacy policy.

Required content:
- Categories of personal data collected and the categories of third parties that receive it.
- How users can review and request changes to their data (if offered).
- How users are notified of policy changes.
- Effective date.
- How the site responds to "Do Not Track" browser signals, and whether third parties track users across sites.

The link must be conspicuous (for example the word "Privacy" in the footer of the home page).

## 3. State comprehensive privacy laws

In force (verify thresholds per state before you conclude a law does not apply):

| State | Effective | Typical threshold |
| --- | --- | --- |
| California (CCPA/CPRA) | 2020 / 2023 | Revenue over USD 26,625,000 (adjusted 1 Jan 2025, next adjustment 2027), or 100,000+ consumers or households, or 50%+ revenue from selling or sharing data |
| Virginia | 2023 | 100,000 consumers, or 25,000 + 50% revenue from data sales |
| Colorado | 2023 | 100,000 consumers, or 25,000 + revenue from sales |
| Connecticut | 2023, amended 1 Jul 2026 | 35,000 consumers (verify) |
| Utah | Dec 2023 | USD 25M revenue + 100,000 consumers |
| Texas | Jul 2024 | No volume threshold. Excludes small businesses as defined by the SBA |
| Oregon | Jul 2024 | 100,000 consumers |
| Montana | Oct 2024 | 50,000 consumers |
| Florida | Jul 2024 | Very large companies only |
| Iowa, Delaware, Nebraska, New Hampshire | Jan 2025 | Varies. Nebraska follows the Texas model |
| New Jersey | Jan 2025 | 100,000 consumers |
| Tennessee | Jul 2025 | USD 25M revenue + 175,000 consumers |
| Minnesota | Jul 2025 | 100,000 consumers |
| Maryland | Oct 2025 | 35,000 consumers. Strict data minimization. Ban on selling sensitive data |
| Indiana, Kentucky | Jan 2026 | 100,000 consumers |
| Rhode Island | Jan 2026 | 35,000 consumers, or 10,000 + 20% revenue from sales |

Common duties when a law applies:
- Privacy notice with categories, purposes, recipients, and rights.
- Rights: access, correction, deletion, portability, opt-out of sale, targeted advertising, and profiling. Usual answer deadline: 45 days.
- Opt-in consent for sensitive data in most states (health, precise location, biometrics, race, religion, sexual orientation, immigration status, data of known children).
- Data Processing Agreements with vendors.
- Data protection assessments for high-risk processing (targeted ads, sale, sensitive data).
- Do not use dark patterns to get consent.

California CPPA regulations (effective 1 Jan 2026): risk assessments (from 2026), rules on automated decisionmaking technology (compliance from 1 Jan 2027), and cybersecurity audits phased by revenue from 2028 (verify dates).

**Startup default:** even below the thresholds, follow the common duties. It costs little at the start, the product will grow, and Texas and Nebraska have no volume threshold.

## 4. Opt-out rights, "Do Not Sell or Share", and GPC

- "Selling" and "sharing" are broad. Sending user data to an ad network through a pixel for targeted ads is usually "sharing" under California law.
- If the business sells or shares data: add a "Do Not Sell or Share My Personal Information" link, or the combined "Your Privacy Choices" link with the official opt-out icon, in the footer.
- If sensitive data is used for more than the allowed purposes: add a "Limit the Use of My Sensitive Personal Information" link.
- **Global Privacy Control (GPC)**: a browser signal that means "opt out". About 12 states require businesses to honor it (California, Colorado, Connecticut, Delaware, Maryland, Minnesota, Montana, Nebraska, New Hampshire, New Jersey, Oregon, Texas) (verify list). Implementation: read `navigator.globalPrivacyControl` in the browser or the `Sec-GPC: 1` request header, and treat it as an opt-out of sale, sharing, and targeted ads for that browser.

## 5. Cookies, pixels, and litigation risk

- No general US law requires opt-in for analytics or ad cookies.
- **But** thousands of lawsuits use the California Invasion of Privacy Act (CIPA, Penal Code 631 and 638.51) and similar state wiretap laws against sites that load tracking pixels, chat widgets, or session replay tools before consent. Courts are split, and the defense cost is high even when the business wins.
- **VPPA** (Video Privacy Protection Act): sites with video content that send video viewing data plus a user identifier to an ad pixel (for example Meta Pixel) face claims of USD 2,500 per violation. The Supreme Court took the case *Salazar v. Paramount Global* (2026) on who counts as a "consumer". Check the result.

**Recommendation:** load no ad pixels and no session replay before consent, even for US users. Get consent for chat recording. Disclose all trackers in the Privacy Policy.

## 6. Minors

- **COPPA** (children under 13): applies to services directed to children, or with actual knowledge of child users. Needs verifiable parental consent before collecting data. The amended COPPA Rule took effect 23 June 2025, with compliance required from 22 April 2026: written data security program, written data retention policy, separate parental consent for disclosure to third parties (for example for ads).
- **Simplest path for a general-audience product:** set a minimum age of 13 (or 18 if the product requires it), add an age gate that does not hint at the "right" answer, and delete data if a child under 13 is detected.
- State laws for teens (age-appropriate design codes, social media age verification, app store age laws in Utah and Texas) are active but many are blocked or partly blocked in court. Check the current status if the product targets teens.
- California: selling data of consumers under 16 needs opt-in consent.

## 7. Terms enforceability (clickwrap)

US courts enforce online terms when the notice is reasonably conspicuous **and** the user takes an action that clearly shows assent.

- Strong: an unchecked checkbox "I agree to the Terms and Conditions" with a visible link, or a button with text right next to it: "By clicking Create account, you agree to the Terms and Privacy Policy" (links in a visible color).
- Weak: a link in the footer only (browsewrap). Small, grey, or far-away text. A button that does not say clicking means agreement (*Berman v. Freedom Financial*, 9th Cir. 2022; *Chabolla v. ClassPass*, 9th Cir. 2025).
- Keep the evidence: which version, when, and how. See `../consent-records.md`.
- Arbitration and class action waivers are generally enforceable in the US if the terms are properly accepted. Mass arbitration risk is real; consider the clause design with a lawyer.

## 8. Refunds

- **No general federal right** to a refund for online purchases.
- **California Civil Code 1723**: a retail seller that does not give full refunds or exchanges within at least 7 days of purchase must post its refund policy clearly. Several other states have similar "posted refund policy" laws (for example New York, Florida, Ohio, Virginia) (verify list).
- **FTC Cooling-Off Rule**: 3 business days to cancel sales made at the buyer's home or a temporary location. Generally does not cover online-only sales.
- Payment card rules and chargebacks: a clear refund policy accepted at checkout helps win disputes.
- The user's 5-day default is legal in the US if it is clearly disclosed before purchase. Laws in other target countries can require more.

## 9. Subscriptions and auto-renewal

- **FTC "Click-to-Cancel" rule**: vacated by the Eighth Circuit on 8 July 2025. Check for new rulemaking.
- **ROSCA** (federal) still applies: clear disclosure of all material terms before billing, express informed consent, and a simple cancel method. The FTC settled with Amazon in September 2025 for USD 2.5 billion over Prime sign-up and cancel flows.
- **California Automatic Renewal Law** (Business and Professions Code 17600 and following, amended by AB 2863, effective 1 July 2025): clear and conspicuous terms near the consent, express affirmative consent to the renewal, an acknowledgment with cancel instructions, online cancel by the same method (a "click to cancel"), annual reminders for yearly plans, notice before a free trial turns paid, and keep proof of consent for at least 3 years or one year after the contract ends, whichever is longer (verify).
- Many other states have auto-renewal laws (for example New York, Colorado, Virginia, Minnesota). Designing to the California standard covers most of them.

## 10. Marketing email and SMS

- **CAN-SPAM** (email): no opt-in required, but: accurate headers and subject, identify ads, include a physical postal address, and a working unsubscribe that is honored within 10 business days. Penalties exceed USD 50,000 per email.
- **TCPA** (calls and SMS): prior express written consent for marketing texts and autodialed calls to mobile phones. Lawsuits are common and damages are USD 500 to 1,500 per message. Use a separate unchecked checkbox with clear text for SMS marketing and store the consent record.
- State "mini-TCPA" laws (Florida, Oklahoma, others) add rules on time and frequency.

## 11. Health data

- **Washington My Health My Data Act**: broad definition of "consumer health data" (it can include fitness, diet, or app usage data that suggests health status). Needs separate consent to collect and separate consent to share, and a separate health data privacy policy. Has a private right of action. Nevada and Connecticut have similar rules.
- **FTC Health Breach Notification Rule**: covers health apps not under HIPAA. Sharing health data without authorization (for example with an ad pixel) counts as a breach.
- **HIPAA**: only if the business is a covered entity or a business associate (it works for a provider, plan, or clearinghouse).

## 12. Data breach notification

All 50 states have breach laws. Deadlines vary: 30 days in several states (for example Colorado, Florida, and California from 2026 (verify)), 45 or 60 days in others, "without unreasonable delay" in the rest. Some require notice to the state Attorney General. The breach plan must list the deadlines for the states where users live.

## 13. Accessibility

- ADA Title III lawsuits against websites and apps of businesses open to the public: thousands per year, mostly in New York, Florida, and California. Courts and settlements use **WCAG 2.1 Level AA** as the standard.
- Recommend WCAG 2.1 AA (or 2.2 AA) from the start, an accessibility statement, and a contact for accessibility problems. Accessibility overlay widgets do not stop lawsuits.

## 14. Copyright (DMCA)

- For products with user content: register a DMCA designated agent in the US Copyright Office online directory to get safe harbor protection. The registration expires after 3 years and must be renewed. Post the agent's contact info on the site and in the Terms. Add a notice and takedown process and a repeat infringer policy.
- Statutory damages for infringement of registered works: up to USD 150,000 per work if willful. See `../media-rights.md`.

## 15. Checklist

- [ ] Privacy Policy that meets CalOPPA, linked in the footer and at sign-up.
- [ ] Check each state law threshold; follow common duties anyway.
- [ ] "Do Not Sell or Share" / "Your Privacy Choices" link if data goes to ad networks.
- [ ] Honor GPC.
- [ ] No pixels or session replay before consent.
- [ ] Minimum age and COPPA decision.
- [ ] Clickwrap: unchecked checkboxes with visible links, consent records.
- [ ] Refund policy clear and accepted at checkout.
- [ ] Subscriptions: California ARL design (consent, acknowledgment, online cancel, records).
- [ ] Separate opt-in for SMS marketing. Unsubscribe in emails with a postal address.
- [ ] Health data: Washington MHMDA check.
- [ ] Breach response plan with state deadlines.
- [ ] WCAG 2.1 AA.
- [ ] DMCA agent if users upload content.
