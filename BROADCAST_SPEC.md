# Catapultor Broadcast — Build Specification

**Status:** Active build spec for Claude Code
**Owner:** Jason (single builder)
**Parent project:** Catapultor (JBW Investments)
**Test property:** Fried Ocean (friedocean.com)
**Long-term vision:** Ship as a product/feature for external creators and properties

---

## 0. Read this first (every session)

Before touching any code in this module, Claude Code must:

1. Read this entire spec.
2. Read `CLAUDE.md` at the repo root for project-wide rules.
3. Read `BROADCAST_CLAUDE.md` at the repo root for module-specific autonomous-mode rules.
4. Check the current phase in `BROADCAST_STATUS.md` to know what's in flight.
5. **Never recreate functionality that already exists in Catapultor or Jason's existing tool stack.** See "Existing assets" section below.
6. **Never modify existing Catapultor features outside the Broadcast module.** Broadcast is additive.
7. Use real Railway + Supabase. No mocks.
8. Deploy backend to Railway. Deploy frontend via `npx vercel --prod`.
9. Never modify credentials or env vars. If one is missing, log as blocker and continue with unblocked work.
10. Commit every 3-5 fixes. Test before commit. Loop until passing.
11. **Operate autonomously.** Do not stop to ask Jason anything. Make decisions, log them in `BROADCAST_STATUS.md`, and keep going until Phase 1a acceptance criteria are met and the smoke test passes.

---

## 1. What Broadcast is

Broadcast is a production pipeline for **user-originated ideas**. 90%+ of articles start with Jason typing an idea seed. The system automates production and iteration from that seed — headlines, images, article body, atomization, platform variants, scheduling, and publishing. Idea recommendation (Opportunity Radar-style) is a secondary, opt-in surface that never auto-populates the article queue or prompts Jason unbidden.

Broadcast turns a single piece of content (a Fried Ocean article idea) into a full multi-channel, monetized distribution campaign with human-in-the-loop approval at the two steps where it matters (headlines and images) and automation everywhere else.

**Core loop:**

Idea seed (user-typed) -> Headline iteration (human) -> Image iteration (human) -> Article + SEO (auto, reviewable) -> Atomization (auto, reviewable) -> Platform variants (auto, reviewable) -> Schedule (auto-proposed, reviewable) -> Publish (auto) -> Analytics (auto)

**What it is not:** another Make scenario. Broadcast replaces Make's fragile glue with proper state management, proper error handling, direct API orchestration, and a UI that keeps Jason in control at the creative moments and out of the way everywhere else. It is also not an idea-generation tool — the ideas come from Jason. Broadcast's job is to produce and distribute them.

---

## 2. Existing assets — USE THESE, DO NOT REBUILD

Jason already has working infrastructure. Broadcast must integrate with, not replace, the following:

### Infrastructure (Catapultor stack — reuse directly)

- **Frontend repo:** `catapultor-frontend` (React, deployed to Vercel)
- **Backend repo:** `catapultor-backend` (Node/Express, deployed to Railway)
- **Database:** Supabase (same instance as Catapultor)
- **GitHub org:** GiddyDonkey
- **Auth + user model:** Whatever Catapultor currently uses — Broadcast reuses it
- **Environment variables:** Already set in Railway for Catapultor — reuse where possible (OpenAI, Supabase, etc.)

### External services already integrated or in active use

| Service | Current role | Broadcast's plan |
|---|---|---|
| **WordPress (friedocean.com)** | Article hosting, already integrated via Application Passwords | Keep. Broadcast publishes articles here via WP REST API. Broadcast is source of truth; WP is a publish target. |
| **Shopify** | Product catalog for Fried Ocean and Catapultor pilots | Keep. Broadcast references products for monetization injection. Does not replace Shopify. |
| **Mailchimp** | Newsletter (~700 subscribers), already integrated and stable | Keep for v1. Broadcast calls Mailchimp API for newsletter sends. Revisit Beehiiv migration at 5k+ subscribers. |
| **Placid** | Templated branded graphics (quote cards, carousel slides, branded social images) | Keep. Broadcast calls Placid API for templated graphic generation where branded templates are needed. |
| **Creatomate** | Templated video (branded intros/outros, lower thirds) | Keep. Broadcast calls Creatomate for templated video work. |
| **Fried Ocean Video Generator** (fried-ocean-video-gen.vercel.app) | AI satirical reel generation — 3 styles (Outrage Bait, Unhinged Absurdist, Dry Deadpan), 1080x1920, FFmpeg-based | Keep. Broadcast integrates this as the primary reel engine. Do NOT rebuild. Call its existing API/endpoint. |
| **Resend** | Transactional email for Catapultor | Keep for Catapultor. Broadcast does not use Resend for newsletter — Mailchimp handles that. |
| **OpenAI** | Already have API key in Railway env | Primary for text (GPT-4o) and image (gpt-image-1). |
| **Ideogram** | Already integrated in Catapultor for design generation | Keep as image fallback provider for Broadcast. |
| **Supabase** | Database + auth | Keep. Broadcast adds new tables under a `broadcast_` prefix to avoid naming collisions with Catapultor tables. |
| **Make scenarios** | Current posting glue (Facebook, YouTube, etc.) | Leave running in parallel during build. Phase out channel-by-channel as Broadcast proves each one. No flag day. |
| **Later** | Fallback buffering/scheduling (low usage, ~1/20 of content) | Phase out once Broadcast has scheduling working. Low priority. |
| **Meta Graph API** | Already used for some posting (Facebook, Instagram) | Extend. Reuse existing app/credentials if possible. Add Threads permissions. |
| **Google Search Console** | Already configured for Fried Ocean | Keep. Broadcast respects existing sitemap/SEO setup. |

### APIs to apply for (Jason handles in parallel — see Section 14)

- TikTok Content Posting API (weeks of approval)
- Pinterest API (days to weeks)
- X API Basic tier ($200/mo — decision pending)
- YouTube Data API quota increase

---

## 3. Architectural principles

1. **Broadcast is the brain; existing tools are the muscles.** Broadcast orchestrates. It does not reimplement what already works.
2. **State machines over cron jobs.** Every article, atom, and scheduled post has a clear state. Transitions are explicit, logged, and recoverable.
3. **Human gates are first-class.** Headlines and images are designed to be iterated on, not just approved. Everything else is auto but reviewable.
4. **Provider abstraction at every external integration.** We will swap providers (especially for image gen) — the abstraction makes it painless.
5. **Prompts are data, not code.** All prompts live in a versioned library in Supabase. Jason edits them in the UI. Prompts are injected with voice profile and context at runtime.
6. **Multi-tenant from day one, single-user in practice.** Workspaces model. Fried Ocean is workspace 1. Future properties slot in cleanly.
7. **Non-destructive parallel run.** Never break what's currently working for Fried Ocean. Broadcast proves itself per-channel before Make scenarios are retired.
8. **Fail loud, recover gracefully.** Tiered alerting: dashboard badges for minor issues, email for critical failures. Every failure is logged with context.
9. **Monetization-aware at every step.** Affiliate link injection, product embedding, newsletter CTAs, tag-based linking are built into the content generation flow, not bolted on.

---

## 4. Data model (Supabase)

All tables prefixed `broadcast_` to coexist with existing Catapultor tables.

### Core tables

**`broadcast_workspaces`**: id (uuid pk), name, slug (unique), owner_user_id (fk to Catapultor users), wordpress_url, wordpress_username, wordpress_app_password_encrypted, shopify_domain (nullable), created_at, updated_at

**`broadcast_voice_profiles`**: id (uuid pk), workspace_id (fk), name, description, tone_rules (jsonb — structured rules: do's, don'ts, vocabulary, sentence length, POV), style_constraints (jsonb), platform_overrides (jsonb), is_active, created_at, updated_at

**`broadcast_voice_examples`** (gold standard library for few-shot): id (uuid pk), voice_profile_id (fk), category (enum: headline, article_intro, social_caption_fb, social_caption_x, social_caption_ig, social_caption_threads, social_caption_bluesky, social_caption_pinterest, video_caption_tiktok, video_caption_yt_shorts, newsletter_blurb, atom_punchline, atom_quote, atom_stat), content (text), source_url (text nullable), performance_tier (enum: gold, silver, bronze), notes, created_at

**`broadcast_prompts`**: id (uuid pk), workspace_id (fk nullable — null = global default), key (text, e.g. headline_ideation), version (int), is_active (boolean — one version per key+workspace active), model (text), system_prompt (text), user_prompt_template (text with {{placeholders}}), parameters (jsonb — temperature, max_tokens), expected_output_format (enum: text, json, markdown, image), created_at, updated_at

**`broadcast_articles`**: id (uuid pk), workspace_id (fk), state (enum: ideation, headline_drafting, headline_locked, image_drafting, image_locked, article_drafting, article_locked, atomized, scheduled, publishing, published, archived, abandoned, parked), idea_seed (text), locked_headline (nullable), locked_subheadline (nullable), article_body (markdown, nullable), seo_title (nullable), seo_description (nullable), seo_keywords (text[] nullable), wordpress_post_id (int nullable), wordpress_url (nullable), published_at (nullable), monetization_config (jsonb), tags (text[]), notes (nullable), parked_reason (nullable), created_at, updated_at

**`broadcast_article_iterations`** (history for headline + image iterations): id (uuid pk), article_id (fk), iteration_type (enum: headline, image_prompt, image_generation), input_context (jsonb), output (text or url), was_selected (boolean), iteration_number (int), model_used (text), cost_cents (int nullable), created_at

**`broadcast_article_assets`**: id (uuid pk), article_id (fk), asset_type (enum: hero_image, social_square, social_vertical, social_landscape, pinterest_pin, carousel_slide, reel_video, short_video, tiktok_video), provider (enum: openai, ideogram, placid, creatomate, fo_video_generator, upload), provider_asset_id (text), url (Supabase storage), width (int), height (int), format (text), generation_prompt (nullable), generation_params (jsonb nullable), is_primary (boolean), created_at

**`broadcast_atoms`**: id (uuid pk), article_id (fk), atom_type (enum: headline, subheadline, punchline, quote, stat, absurd_detail, image_reference, callout), content (text), hero_asset_id (fk nullable), sort_order (int), is_active (boolean), created_at

**`broadcast_platform_variants`**: id (uuid pk), atom_id (fk), platform (enum: facebook, instagram_feed, instagram_reel, threads, x, bluesky, pinterest, tiktok, youtube_shorts, newsletter, wordpress), copy (text), hashtags (text[]), mentions (text[]), cta (text nullable), asset_id (fk nullable), generation_prompt_key (text), is_approved (boolean), approved_at (nullable), edited_by_user (boolean), created_at, updated_at

**`broadcast_scheduled_posts`**: id (uuid pk), platform_variant_id (fk), workspace_id (fk denormalized), scheduled_for (timestamp), state (enum: scheduled, queued, publishing, published, failed, cancelled), timezone (text), publish_attempts (int default 0), last_attempt_at (nullable), published_at (nullable), platform_post_id (nullable), platform_post_url (nullable), error_log (jsonb[] nullable), created_at, updated_at

**`broadcast_platform_connections`**: id (uuid pk), workspace_id (fk), platform (enum), account_handle (text, e.g. @friedocean), account_id (text), access_token_encrypted, refresh_token_encrypted (nullable), token_expires_at (nullable), token_last_refreshed_at (nullable), permissions (text[]), is_healthy (boolean), last_health_check_at (nullable), metadata (jsonb), created_at, updated_at

**`broadcast_post_analytics`**: id (uuid pk), scheduled_post_id (fk), platform (enum), fetched_at (timestamp), impressions (nullable), reach (nullable), engagements (nullable), likes (nullable), comments (nullable), shares (nullable), saves (nullable), clicks (nullable), watch_time_seconds (nullable), completion_rate (float nullable), raw_payload (jsonb)

**`broadcast_monetization_links`**: id (uuid pk), workspace_id (fk), link_type (enum: affiliate, shopify_product, newsletter_cta, external), label (text), destination_url (text), utm_parameters (jsonb), provider (text nullable, e.g. amazon, shopify), provider_metadata (jsonb nullable), is_active (boolean), created_at, updated_at

**`broadcast_link_clicks`**: id (uuid pk), monetization_link_id (fk), article_id (fk nullable), platform (nullable), clicked_at (timestamp), referrer (nullable)

**`broadcast_alerts`**: id (uuid pk), workspace_id (fk), severity (enum: info, warning, error, critical), category (enum: token_expiring, token_expired, publish_failed, api_down, quota_exceeded, generation_failed, connection_unhealthy, needs_approval), message (text), context (jsonb), related_entity_type (text nullable), related_entity_id (uuid nullable), is_resolved (boolean), resolved_at (nullable), email_sent_at (nullable), created_at

**`broadcast_competitor_sources`** (scaffold for phase 2): id (uuid pk), name (text, e.g. "The Onion"), url (text), feed_url (nullable — RSS), is_active (boolean), last_scraped_at (nullable)

**`broadcast_competitor_content`** (scaffold for phase 2): id (uuid pk), source_id (fk), url (unique), headline, published_at (nullable), excerpt (nullable), topic_tags (text[]), scraped_at

### Critical indexes

- `broadcast_articles (workspace_id, state, updated_at desc)`
- `broadcast_scheduled_posts (scheduled_for, state)` — scheduler queries
- `broadcast_platform_connections (workspace_id, platform)` — unique
- `broadcast_alerts (workspace_id, is_resolved, severity, created_at desc)`
- `broadcast_post_analytics (scheduled_post_id, fetched_at desc)`

---

## 5. Service layer architecture

All services live in `catapultor-backend/src/broadcast/`.

### 5.1 Orchestration engine (`src/broadcast/orchestrator/`)

Drives article state machine. Exposes: `advanceArticle(articleId, targetState, context)`, `parkArticle(articleId, reason)`, `abandonArticle(articleId, reason)`, and an `onStateChange` event bus.

Allowed transitions: ideation -> headline_drafting -> headline_locked -> image_drafting -> image_locked -> article_drafting -> article_locked -> atomized -> scheduled -> publishing -> published. Plus: any -> abandoned. Plus: any (pre-published) -> parked (reversible).

### 5.2 Prompt service (`src/broadcast/prompts/`)

- `getPrompt(key, workspaceId)` — active version, falls back to global
- `renderPrompt(promptId, context, voiceProfileId)` — injects voice profile + few-shot examples
- `runPrompt(promptId, context, voiceProfileId, options)` — renders + calls model, logs cost
- `testPrompt(promptId, sampleContext)` — UI-driven test harness
- `createVersion(promptId, changes)` — version bumping, old versions retained

Voice profile injection: load structured voice profile (rules, constraints), select N few-shot examples from `broadcast_voice_examples` where category matches prompt's output category, weighted by performance_tier (gold > silver > bronze). Render examples into system prompt. Apply platform overrides if prompt is platform-specific.

### 5.3 Image service (`src/broadcast/images/`) — THE HARDEST PART

Multi-stage pipeline:

1. **Brief generation** (Claude / GPT-4o): Given article headline + body excerpt + voice profile, produce structured JSON: subject, setting, composition (landscape, centered, rule of thirds), mood, style (photorealistic documentary), lighting, avoid (text, logos, distorted faces, watermarks), aspect_ratio.

2. **Prompt construction** (deterministic templating): Convert brief into provider-optimized prompt using a library of proven pattern fragments stored under prompt keys like `image_style_photorealistic`, `image_constraint_no_text`, etc. These encode Jason's successful manual prompt patterns (seed with best guesses and iterate).

3. **Generation**: Primary OpenAI `gpt-image-1`, fallback Ideogram. Interface: `generateImage(finalPrompt, size, quality, provider) -> {url, assetId, cost}`.

4. **Quality check**: auto (no image, wrong aspect ratio, corrupt) + human review via iteration UI.

5. **Iteration UI controls**: regenerate same prompt (new seed), adjust brief, keep subject change style, keep composition change mood, more absurd / less absurd preset modifiers, provider swap, upload manual override.

6. **Asset variants**: Once hero locked, auto-generate other format variants (square, vertical, Pinterest) via OpenAI re-render or intelligent cropping.

### 5.4 Atomization service (`src/broadcast/atomizer/`)

Input: locked article. Output: set of atoms proposed for review. Claude prompt produces JSON array of atoms with type, content, rationale. Jason reviews in UI, approves/edits/rejects individually.

### 5.5 Platform variant service (`src/broadcast/variants/`)

For each approved atom x platform, call platform-specific prompt (keyed like `variant_x`, `variant_threads`, etc.) producing platform-appropriate copy (character limits enforced), hashtags, CTA, asset selection.

Staggering logic for X / Threads / Bluesky: service proposes schedule grid per article using configurable cadence rules per platform (e.g., "X: 3 atoms over 5 days, spaced 1.5 days apart, starting at launch").

### 5.6 Distribution service (`src/broadcast/distribution/`)

Common interface: `publishPost(scheduledPostId) -> {success, platformPostId, platformPostUrl, error?}`

Adapters in `src/broadcast/distribution/adapters/`:
- `metaAdapter.js` — Facebook, Instagram Feed, Instagram Reels, Threads
- `youtubeAdapter.js` — YouTube Shorts, regular YouTube
- `tiktokAdapter.js` — TikTok Content Posting
- `xAdapter.js` — X API v2
- `blueskyAdapter.js` — AT Protocol
- `pinterestAdapter.js` — Pinterest API
- `wordpressAdapter.js` — WP REST via Application Passwords
- `mailchimpAdapter.js` — Mailchimp campaigns API

Each adapter implements the common interface and handles platform quirks (rate limits, media upload flow, retry).

### 5.7 Token manager (`src/broadcast/tokens/`)

- Daily cron refreshes anything expiring within 7 days
- Health check cron pings each connection's API, updates is_healthy
- On refresh failure: `token_expired` alert
- UI at `/broadcast/connections`
- All tokens encrypted at rest

### 5.8 Scheduler (`src/broadcast/scheduler/`)

- Cron every minute
- Queries scheduled_posts where scheduled_for <= now AND state = scheduled
- Sets queued, hands to distribution service
- On failure: exponential backoff retries up to N, then failed + critical alert

### 5.9 Analytics ingester (`src/broadcast/analytics/`)

- Hourly first 24h post-publish, daily for 7 days, weekly for 30 days
- Per-platform fetchers -> `broadcast_post_analytics` with raw_payload

### 5.10 Alerting service (`src/broadcast/alerts/`)

Tiers: info (badge), warning (badge + yellow), error (red + email after 1h), critical (red + immediate email). Email via Resend.

### 5.11 Monetization service (`src/broadcast/monetization/`)

Link library CRUD, UTM auto-generation per article/platform/atom, click tracking via redirect endpoint (`broadcast.catapultor.com/go/:linkId?article=X&platform=Y`), attribution report.

### 5.12 Competitor service scaffold (`src/broadcast/competitors/`) — phase 2

Tables created in phase 1, scrapers + UI deferred. Data model ready.

---

## 6. Frontend structure (`catapultor-frontend/src/broadcast/`)

Screens: `/broadcast` (dashboard), `/broadcast/articles/new` (start article), `/broadcast/articles/:id` (workflow wizard), `/broadcast/articles/:id/headline` (headline iteration), `/broadcast/articles/:id/image` (image iteration), `/broadcast/articles/:id/atomize` (atomization review), `/broadcast/articles/:id/distribute` (platform variants + schedule), `/broadcast/schedule` (schedule grid), `/broadcast/drafts` (parked articles), `/broadcast/prompts` (prompts library), `/broadcast/voice` (voice profile editor), `/broadcast/analytics` (analytics dashboard), `/broadcast/connections` (platform connections), `/broadcast/monetization` (monetization panel), `/broadcast/alerts` (alerts log).

Design direction: command center feel — dense but organized, data-rich, clear visual hierarchy. Dark theme default. Editorial typography to match Fried Ocean voice — confident serif or distinctive sans for headers, clean mono or humanist sans for body/data. Status colors used sparingly (green = published, amber = needs attention, red = failed). Subtle state-transition animations.

---

## 7. Prompt library initial seed set

Seed these in `broadcast_prompts` on first setup:

`headline_ideation`, `headline_refinement`, `image_brief_builder`, `image_prompt_builder_openai`, `image_prompt_builder_ideogram`, `article_body`, `article_seo_meta`, `atomization`, `variant_facebook`, `variant_instagram_feed`, `variant_instagram_reel`, `variant_threads`, `variant_x`, `variant_bluesky`, `variant_pinterest`, `variant_tiktok`, `variant_youtube_shorts`, `variant_newsletter_blurb`, `schedule_proposer`.

Each ships with initial system + user templates. Best-guess content is fine for v1; Jason refines via UI post-launch.

---

## 8. Phase 1 build sequence

### Phase 1a — Foundation + Tier 1 (THIS PHASE, autonomous execution)

**Week 1 — Scaffolding**
- Create `src/broadcast/` in both repos
- Run all Supabase migrations (Section 4)
- Workspaces CRUD, seed Fried Ocean as workspace 1
- Voice profiles + examples CRUD
- Prompts library service + seed initial prompts
- `/broadcast` dashboard shell, `/broadcast/prompts`, `/broadcast/voice` screens
- Auth integration: behind existing Catapultor auth

**Week 2 — Article workflow core**
- Orchestration engine + state machine
- Article creation flow (`/broadcast/articles/new`)
- Headline iteration loop (`/broadcast/articles/:id/headline`)
- Image brief + generation + iteration loop (`/broadcast/articles/:id/image`)
- OpenAI gpt-image-1 primary, Ideogram fallback
- Proven-pattern fragment library for image prompts

**Week 3 — First publish path**
- Article generation (auto) + review
- SEO metadata generation
- WordPress adapter + publish flow
- Facebook adapter + publish flow
- Mailchimp adapter + newsletter send flow
- Basic schedule grid
- Scheduler cron
- Meta token manager
- Basic dashboard alerting
- End-to-end smoke test: idea -> headline -> image -> article -> WP + FB + newsletter published

### Phase 1b — Video + Tier 2 (after Phase 1a deployed)
Integrate FO Video Generator + Creatomate, YouTube Shorts adapter, TikTok adapter (when API approved), video variant management UI.

### Phase 1c — Atomization + Tier 3 (after 1b)
Atomization service + review UI, variants for IG / Bluesky / X / Threads / Pinterest, staggered posting, Placid templated graphics, monetization panel + click tracking, email alerting, basic analytics for Meta + YouTube.

### Phase 1d — Archive migration + polish
WP archive scraper -> backfill articles, backfill voice examples from top past posts, full analytics dashboard, competitor scraping scaffold.

---

## 9. Non-negotiable quality requirements

1. Image quality matches Jason's manual ChatGPT output. Iterate until true.
2. Voice consistency across every generated output. Voice profile + few-shot always injected.
3. No content publishes without explicit approval unless Jason enables full-auto per workspace per platform.
4. Every failure logged and surfaced. No silent failures.
5. Existing Fried Ocean operations never break during build.

---

## 10. Deliberately deferred

Multi-user collaboration, paid newsletter tier, Beehiiv migration, full competitor UI (phase 2), ads module (phase 3), shop module (phase 4), multi-timezone scheduling, Amazon PA API automation (blocked on qualifying sales).

---

## 11. Cost awareness

Every article = potentially hundreds of API calls. Log cost_cents on every model call. Surface per-article cost in UI. Target <$2/article steady state.

---

## 12. Testing strategy

Unit tests on state machine transitions. Integration tests on prompt rendering with voice injection. Integration tests on each platform adapter (sandbox where possible). End-to-end smoke test through WP + FB + newsletter. Run tests before every commit.

---

## 13. Claude Code operating rules — AUTONOMOUS MODE

Full rules in `BROADCAST_CLAUDE.md`. Summary:

1. Autonomous by default. Do not stop, ask, or pause. Make decisions, log them, keep going.
2. Read `BROADCAST_SPEC.md`, `BROADCAST_CLAUDE.md`, `BROADCAST_STATUS.md`, root `CLAUDE.md` at start of session.
3. Update `BROADCAST_STATUS.md` at end of session.
4. Never recreate existing tooling — integrate.
5. Never modify non-Broadcast Catapultor code.
6. Real Railway + Supabase. No mocks.
7. Commit every 3-5 fixes with `[Broadcast]` prefix. Push to remote.
8. Deploy backend to Railway, frontend via `npx vercel --prod`.
9. Test as you go. Run before commit. Fix failures immediately.
10. On failure, loop until passing — never leave broken code.
11. When decision isn't specified, pick option most aligned with Section 3 principles, log under "Decisions made autonomously", continue.
12. All new tables prefixed `broadcast_`. All API routes `/api/broadcast/`. All frontend routes `/broadcast/`.
13. Image quality bar: matches Jason's manual ChatGPT output. Iterate until met.
14. Never modify credentials. If missing, log blocker, continue with unblocked work.
15. Use non-interactive flags on all CLI commands. Never hang on a prompt.

Only reasons to stop a session:
- Missing credential you cannot work around (log, continue with unblocked work, end when unblocked work done)
- External API genuinely down (not your bug)
- All Phase 1a acceptance criteria met and smoke test passes

---

## 14. Jason's parallel action items (outside Claude Code's scope)

- Apply for TikTok Content Posting API access
- Apply for Pinterest API access
- Decide on X API Basic tier ($200/mo) and activate if yes
- Apply for YouTube Data API quota increase
- Verify Meta app permissions include Threads + IG Content Publishing
- Curate initial 20-30 gold standard Fried Ocean examples
- Document current manual ChatGPT image prompts that produce best results
- Note platform handles, hashtag conventions, account-specific prefs

---

## 15. Success criteria for Phase 1 complete

1. Jason can go from idea seed to fully distributed campaign (article + hero image + atoms across all target platforms + newsletter) without touching Make, Later, or any existing tool — output quality >= current manual process
2. All platform tokens auto-refresh; failures surface as alerts
3. Scheduling grid reflects everything in flight; drag-to-reschedule works
4. Image service consistently produces article-ready hero images without manual override
5. Analytics flows back in for Meta + YouTube
6. Make scenarios retired for all channels Broadcast has replaced
7. Architecture clean enough that adding a second workspace requires no code changes

**End of spec.**
