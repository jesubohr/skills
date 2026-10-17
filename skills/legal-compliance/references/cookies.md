# Cookies Policy and Cookie Consent

## Step 1: Inventory

List every cookie, local storage key, session storage key, SDK identifier, and tracking pixel. Get them from:

- The code (search for `document.cookie`, `localStorage`, `setCookie`, analytics and ads SDK setup).
- Third-party scripts in HTML templates and tag managers.
- A real browser session: open the site with empty storage and list cookies and network requests to third-party domains before any interaction.

For mobile apps, the same logic applies to SDKs that collect device identifiers (advertising ID, IDFA). On iOS, tracking across apps also needs the App Tracking Transparency prompt.

For each item, record: name, provider (first or third party), purpose, category, duration, and if it shares data with a third party.

## Step 2: Categorize

| Category | Examples | Needs consent? |
| --- | --- | --- |
| Strictly necessary | Session, login, CSRF, load balancer, cart, the consent choice itself | No. Inform only |
| Preferences | Language, theme | Usually no if the user set it. Inform |
| Analytics | Google Analytics, Mixpanel, PostHog, Hotjar | Depends on country. See below |
| Marketing and ads | Meta Pixel, Google Ads, TikTok Pixel, LinkedIn Insight, retargeting | Opt-in in strict countries. Opt-out rights in the US |
| Session replay and chat recording | Hotjar, FullStory, LogRocket, Clarity | Treat as high risk in all countries |

## Step 3: Decide if a consent banner is needed

Decide based on the target countries:

- **European Union, UK, Brazil (ANPD guidance):** prior opt-in for all non-necessary cookies. A real banner with "Accept", "Reject" with equal visibility, and "Settings". No trackers before consent.
- **Most of Latin America (Colombia, Mexico, Argentina, Chile, Peru):** cookies that identify a person are personal data. The law requires information and, in general, consent for non-necessary purposes. An opt-in banner is the safe default. Check `jurisdictions/latam.md` for current guidance.
- **United States:** no general opt-in rule, but: state privacy laws require an opt-out of "sale/sharing" and targeted ads (a "Do Not Sell or Share My Personal Information" link), and honoring the Global Privacy Control (GPC) browser signal. Tracking pixels and session replay without consent create lawsuit risk under wiretap laws (for example California CIPA) and the Video Privacy Protection Act if there is video. See `jurisdictions/united-states.md`.

**Simplest compliant default for a startup that targets several countries:** use one opt-in banner for all users, honor GPC, and load no analytics or marketing script before consent. If the product has no analytics or marketing cookies at all, no banner is needed; a cookie notice in the Privacy or Cookies Policy is enough. Recommend privacy-friendly, cookie-less analytics when the user does not need ad tracking. This removes the banner and much of the risk.

## Step 4: Technical rules for the banner

- Block non-necessary scripts until consent. Load them only after the matching category is accepted. Most violations come from scripts in the HTML head that load before the banner.
- "Reject" must be as easy as "Accept" (same level, same visual weight) where opt-in is required.
- Store the choice and the banner version. See `consent-records.md`.
- Let the user change their choice at any time (a "Cookie settings" link in the footer).
- Ask again when the cookie list changes in a material way, or after a set period (for example 12 months).
- If a consent management platform (CMP) is used, check that it is configured to block scripts, not only to show a banner.

## Cookies Policy content

Can be a separate page or a section in the Privacy Policy. Include:

1. What cookies and similar technologies are, in plain words.
2. The table from Step 1 (name, provider, purpose, category, duration).
3. Legal basis for each category.
4. How to accept, reject, and change choices, and how to delete cookies in the browser.
5. Third parties and links to their policies.
6. International transfers caused by third-party cookies.
7. Version and last updated date.
