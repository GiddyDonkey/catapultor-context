# Catapultor Backlog

Running log of shipped, in-flight, and deferred work for Catapultor (including the Broadcast module). Companion to `BROADCAST_STATUS.md`, which is scoped to the live Phase 1a Broadcast build.

## Done

- **Legal & compliance pages** (2026-04-16) — built and deployed to https://www.catapultor.com
  - [x] Privacy Policy — https://www.catapultor.com/privacy
  - [x] Terms of Service — https://www.catapultor.com/terms
  - [x] About Catapultor — https://www.catapultor.com/about
  - [x] Contact — https://www.catapultor.com/contact
  - [x] Data Deletion (required by Meta app review) — https://www.catapultor.com/data-deletion
  - [x] Site-wide footer with compliance links on every page including the landing page

  Platform applications (Pinterest, TikTok, YouTube, Meta when submitted) should use the URLs above in their developer portal "Privacy Policy URL", "Terms of Service URL", and "Data Deletion URL" fields.

## Broadcast Backlog

### P0

- [ ] **BRD-007: Natural language iteration** — Allow Jason to type free-text feedback on any generation output (headline, image, article body) and have the system re-generate incorporating that feedback. This is where compounding value lives — each iteration gets closer to the target with less effort. Applies to headline_refinement (already built), image regeneration (brief adjustment), and article body (rewrite with notes).

### P1

- [ ] **BRD-080: Promote idea seed input to primary dashboard CTA** — DONE (2026-04-16). Idea seed text area is now the dominant element on /broadcast dashboard with optional structured fields (tone hint, must-include, avoid). These pipe into headline_ideation context.

### P2

- [ ] **BRD-081: Move idea recommendations to /broadcast/inspiration** — When Opportunity Radar or trending topic features are built, they must live at /broadcast/inspiration as an opt-in browse surface. They must NOT auto-populate the article queue, auto-create articles, or prompt Jason unbidden. Broadcast is a production pipeline for user-originated ideas — recommendations are secondary and opt-in.

### P3

- [ ] **BRD-082: Pinterest privacy policy URL update** — Jason manual task: update Pinterest developer app (ID 1562567) at developers.pinterest.com to use https://www.catapultor.com/privacy instead of the temporary GitHub URL. Same for any other platform dev portal submitted with temporary URLs.

## DBT (Deferred / Blocked / Todo)

- [ ] **Replace business mailing address placeholder in /contact** with the real address when the JBW Investments LLC mailing address is finalized. The current page says "Business mailing address coming soon." in the `Mailing address` section of `src/pages/legal/Contact.js`.
