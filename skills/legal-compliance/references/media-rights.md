# Image and Media Rights

Using an image, video, font, icon, or audio file without a license can cause takedown notices, app store removal, and claims for damages. In the US, statutory damages for registered works can reach USD 150,000 per work for willful infringement. Photo agencies actively scan the web for unlicensed use.

## Method

1. List every media asset:
   - Files in the repo (images, videos, audio, fonts, icons, illustrations, 3D models, Lottie files).
   - Assets loaded from URLs or a CMS.
   - Fonts loaded from CDNs.
   - Assets in app store listings, social media, ads, and emails, if in scope.
2. For each asset, find the source and the license. Ask the user when the source is unknown.
3. Classify the status:
   - **OK**: own work, or a license that covers this use, with proof stored.
   - **Check**: license exists but the terms may not cover this use (for example commercial use, logo use, merchandise, editorial-only, attribution missing).
   - **Replace**: unknown source, copied from the internet or a search engine, or no proof.
4. Write the result to `media-inventory.md`.

## Rules to check

- "Found on Google" is not a license. Replace it.
- **Stock sites**: check the license type (standard vs extended), the number of users or copies, and if the use is commercial. "Editorial use only" images cannot be used in marketing or product UI.
- **Free sites** (Unsplash, Pexels, Pixabay): usually free for commercial use, but check the current terms. Identifiable people, trademarks, and buildings can still need separate permission.
- **Creative Commons**: CC BY needs attribution. NC (non-commercial) is not allowed in a commercial product. ND (no derivatives) does not allow edits.
- **Fonts**: the font license must cover web embedding and app embedding. Google Fonts are usually under the SIL Open Font License. Commercial fonts often need a separate web or app license. Also check the privacy aspect: loading fonts from a third-party CDN sends the visitor IP to that provider. Self-hosting avoids this.
- **Icons**: check the icon set license (MIT, CC BY, commercial). Some free sets require attribution.
- **AI-generated media**: check the terms of the AI tool for commercial use. In many countries, purely AI-generated content may not get copyright protection, so the business cannot stop others from copying it. Do not generate images of real people or trademarks.
- **Work from designers or freelancers**: the business needs a written contract that assigns the rights (or grants a broad license). Without it, the creator often keeps the copyright. In Latin America, moral rights cannot be transferred, but economic rights can be.
- **Photos of people**: a written model release (consent to use their image) is needed for commercial use. Image of a person is personal data in many Latin American countries.
- **Third-party logos and trademarks**: "As seen in" or "Our clients" sections need permission from each company.
- **Music and video**: background music needs a sync license. Embedded videos from YouTube or Vimeo are fine to embed, but they load trackers (see the cookies reference). Use privacy-enhanced embed modes.
- **Open source code and assets**: list licenses. Some (for example GPL) have duties when you distribute the software.

## Store the proof

Recommend a folder or a shared drive with license receipts, contracts, and releases, named so they match the `media-inventory.md` rows. If a claim arrives years later, the proof must still exist.

## User-generated content

If users upload content, the Terms must:

- Say the user owns or has the rights to what they upload.
- Grant the business a license to host, display, and process it for the service.
- Include a takedown process for copyright claims. In the US, register a DMCA designated agent with the Copyright Office to get safe harbor protection (renew every 3 years).

## media-inventory.md format

| Asset | Location in project | Type | Source | License | Proof location | Attribution needed? | Status | Action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
