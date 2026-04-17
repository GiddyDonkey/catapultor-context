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

- [ ] **BRD-007: Natural language iteration** — Allow Jason to type free-text feedback on any generation output (headline, image, article body) and have the system re-generate incorporating that feedback. Applies to headline_refinement, image regeneration, and article body. | agent_safe: no

### P1

- [x] **BRD-080: Promote idea seed input to primary dashboard CTA** — DONE (2026-04-16). | agent_safe: no
- [ ] **BRD-090: Unit tests for broadcast state machine integration** — Write comprehensive integration tests for the orchestrator: create article, advance through all states, verify DB state at each transition, test park/unpark/abandon, test invalid transitions. Directory: src/broadcast/tests/. | status: scoped | agent_safe: yes
- [ ] **BRD-091: Unit tests for prompt service** — Expand test coverage: test getPrompt workspace-scoped vs global fallback, test createVersion with proper version numbering, test renderPrompt placeholder substitution with missing keys, test few-shot injection counts. Directory: src/broadcast/tests/. | status: scoped | agent_safe: yes
- [ ] **BRD-092: Unit tests for distribution adapters** — Write adapter interface tests: verify each adapter exports publishPost/publishDirect, test error handling when credentials missing, test retry logic in distributionService. Mock external HTTP calls. Directory: src/broadcast/tests/. | status: scoped | agent_safe: yes

### P2

- [ ] **BRD-081: Move idea recommendations to /broadcast/inspiration** — opt-in browse surface, not auto-populating. | agent_safe: no
- [ ] **BRD-093: Add CHANGELOG.md** — Create a conventional CHANGELOG.md from git history. Include all [Broadcast] commits grouped by date. Directory: root. | status: scoped | agent_safe: yes
- [ ] **BRD-094: Lint + import cleanup across src/broadcast/** — Run ESLint on all broadcast backend files, fix auto-fixable issues, sort imports alphabetically. No logic changes. Directory: src/broadcast/. | status: scoped | agent_safe: yes

### P3

- [ ] **BRD-082: Pinterest privacy policy URL update** — Jason manual task. | agent_safe: no
- [ ] **BRD-095: Add JSDoc comments to all broadcast service exports** — Document all exported functions in orchestrator, prompts, images, distribution, scheduler, alerts, tokens services with JSDoc @param, @returns, @throws. No logic changes. Directory: src/broadcast/. | status: scoped | agent_safe: yes

## DBT (Deferred / Blocked / Todo)

- [ ] **Replace business mailing address placeholder in /contact** with the real address when the JBW Investments LLC mailing address is finalized. | agent_safe: no
