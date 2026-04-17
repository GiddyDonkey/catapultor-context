# BROADCAST_STATUS.md — Current state of the Broadcast build

**Last updated by:** Claude Code — 2026-04-16 (session 3 — five fixes)
**Operating mode:** AUTONOMOUS — keep working until Phase 1a acceptance criteria are met and the end-to-end smoke test passes. No stopping to ask.

## Current phase

Phase 1a — Foundation + Tier 1 (execute continuously)

## Existing assets inventory (Step 0 output — populated at kickoff)

**GitHub repos (confirmed accessible):**
- catapultor-backend: https://github.com/GiddyDonkey/catapultor-backend.git (branch: main)
- catapultor-frontend: https://github.com/GiddyDonkey/catapultor-frontend.git (branch: main)

**Railway (existing services + env var names reused by Broadcast):**
- Service: catapultor-backend (deployed at catapultor-backend-production-c071.up.railway.app)
- Env vars available for reuse: OPENAI_API_KEY, SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, SUPABASE_ANON_KEY, IDEOGRAM_API_KEY, PLACID_API_KEY, RESEND_API_KEY, META_APP_ID, META_APP_SECRET, BACKEND_URL, CORS_ORIGIN

**Vercel (existing project + env var names reused):**
- Project: catapultor-frontend (deployed at www.catapultor.com)
- Env vars available for reuse: REACT_APP_API_URL (points to Railway backend)

**Supabase (existing schema references):**
- Project ref: ilrmcnifkdpowsoryiam
- Catapultor auth: uses Supabase Auth (supabase.auth.getUser), user table is auth.users
- Catapultor stores table: `stores` (stores.user_id FK to auth.users.id)
- Service role key: available via SUPABASE_SERVICE_ROLE_KEY env var

**External integrations — confirmed operational:**
- OpenAI (API key in env var: OPENAI_API_KEY) — confirmed, used throughout Catapultor
- Ideogram (API key in env var: IDEOGRAM_API_KEY) — confirmed, used for brand marks
- Placid (API key in env var: PLACID_API_KEY) — confirmed but deprecated in Catapultor
- Resend (API key in env var: RESEND_API_KEY) — confirmed, used for transactional email
- Meta app credentials (env vars: META_APP_ID, META_APP_SECRET) — app exists but noted as "credential-blocked" for page tokens
- Shopify (env vars: SHOPIFY_CLIENT_ID, SHOPIFY_CLIENT_SECRET, SHOPIFY_ADMIN_ACCESS_TOKEN)

**External integrations — NOT YET IN ENV VARS (blockers for end-to-end smoke test):**
- WordPress Application Password for friedocean.com — NOT in Railway env vars. Need: BROADCAST_WP_URL, BROADCAST_WP_USERNAME, BROADCAST_WP_APP_PASSWORD
- Mailchimp API key — NOT in Railway env vars. Need: BROADCAST_MAILCHIMP_API_KEY, BROADCAST_MAILCHIMP_AUDIENCE_ID
- Meta/Facebook page access token — Meta app credentials exist but no page-level access token for Fried Ocean FB page. Need: BROADCAST_META_PAGE_ACCESS_TOKEN, BROADCAST_META_PAGE_ID
- Creatomate — NOT in Railway env vars
- FO Video Generator — endpoint known (fried-ocean-video-gen.vercel.app) but no API key env var

**Backend architecture notes:**
- Backend has NO `src/` directory — code lives at root (routes/, services/, middleware/, utils/)
- Entry point: index.js (Express app)
- Auth: Supabase Auth via middleware/auth.js (requireAuth, requireStoreAccess)
- Package format: CommonJS (require/module.exports)
- Deploy: git push to main triggers Railway auto-deploy

**Frontend architecture notes:**
- React CRA with src/ directory
- React 19, react-router-dom v7, styled-components, axios
- Entry point: src/App.js with lazy-loaded routes
- Auth: localStorage catapultor_token with ProtectedRoute wrapper
- API client: src/api.js (axios with interceptors)
- Deploy: `npx vercel --prod`

## Progress

### Completed
- [x] Inventory existing assets (GitHub, Railway, Vercel, Supabase)
- [x] Created BROADCAST_SPEC.md, BROADCAST_CLAUDE.md, BROADCAST_STATUS.md in both repos
- [x] Create `src/broadcast/` directory structure in `catapultor-backend`
- [x] Create `src/broadcast/` directory structure in `catapultor-frontend`
- [x] Write and run Supabase migration — 17 broadcast_ tables live with indexes and triggers
- [x] Seed `broadcast_workspaces` with Fried Ocean as workspace 1 (ID: 977fa32c-8d64-48bb-a2e1-5d2e42778414, user: 29@friedocean.com)
- [x] Seed voice profile with placeholder examples (8 examples across headline, social, newsletter, atom categories)
- [x] Seed `broadcast_prompts` with all 18 initial prompts (headline_ideation through schedule_proposer)
- [x] Implement workspaces + voice profiles + prompts services
- [x] Build prompt-runner service (voice injection, few-shot, model call, cost logging)
- [x] Implement orchestration engine + article state machine
- [x] Build image service (OpenAI gpt-image-1 primary, Ideogram fallback, multi-stage pipeline)
- [x] Implement WordPress adapter (WP REST API + Application Passwords)
- [x] Implement Facebook adapter (Meta Graph API)
- [x] Implement Mailchimp adapter (campaign creation + send)
- [x] Build distribution service (common interface with retry + alerting)
- [x] Build scheduler service (1-min interval cron for due posts)
- [x] Build alert service (tiered: info/warning/error/critical with Resend email)
- [x] Build all backend API routes at /api/broadcast/* with auth
- [x] Mount broadcast routes in index.js + start scheduler
- [x] Build `/broadcast` dashboard shell (dark theme, stats, article list, alerts)
- [x] Build `/broadcast/articles/new` — article creation from idea seed
- [x] Build `/broadcast/articles/:id` — 5-step workflow wizard (headline → image → article → review → publish)
- [x] Build `/broadcast/prompts` — prompts library editor with test harness
- [x] Build `/broadcast/voice` — voice profile editor with example management
- [x] Build `/broadcast/schedule` — monthly calendar grid
- [x] Build `/broadcast/connections` — platform connections with health check
- [x] Deploy backend to Railway (auto-deploy via git push) — /api/broadcast/health returns OK
- [x] Deploy frontend to Vercel — accessible at www.catapultor.com/broadcast

- [x] Write unit tests: 28 state machine assertions + 5 prompt service integration tests (all passing)
- [x] Full generation pipeline test: idea -> 10 headlines -> hero image -> article body -> SEO -> 12 atoms -> 2 platform variants
- [x] Fix image persistence: all generated images now uploaded to Supabase Storage (ephemeral provider URLs expire)
- [x] Multi-name env var fallback for platform credentials (BROADCAST_*, WP_*, WORDPRESS_*, META_*, FB_*, MAILCHIMP_*)
- [x] Credential health check diagnostic endpoint at /api/broadcast/health/credentials
- [x] **FIX P1:** Auth — Jason's account found ([REDACTED].com, id: 9d133584), password reset, workspace ownership transferred. ProtectedRoute now redirects to /login?returnTo= for /broadcast/* routes.
- [x] **FIX P2:** Credential diagnostic — simplified to direct BROADCAST_* name checks. Diagnostic is accurate: vars genuinely not present on Railway (confirmed with envVarStatus showing all false).
- [x] **FIX P3:** OpenAI image gen — root cause was invalid size 1792x1024 (valid: 1024x1024, 1024x1536, 1536x1024). Fixed to 1536x1024. Added 5 hard quality constraints to every prompt. Added GPT-4o vision quality gate with 3-attempt retry. Regenerated hero image: PASSED quality gate on first attempt.
- [x] **FIX P4:** Blank /broadcast/articles — added redirect to /broadcast. Added ComingSoon placeholders for /broadcast/drafts, /broadcast/monetization, /broadcast/alerts.

### In flight
- [ ] Platform credential verification on Railway (env vars not present — Jason action required)

### Next up (execute in order)
1. **Jason action:** Verify BROADCAST_* env vars are actually set in Railway on the catapultor-backend service. Diagnostic URL: https://catapultor-backend-production-c071.up.railway.app/api/broadcast/health/credentials shows all 7 as false.
2. Credential auth verification passes for all 3 platforms
3. Jason reviews test article in dashboard at https://www.catapultor.com/broadcast/articles/52bb5195-7374-46df-b21c-61fac8cf364d
4. End-to-end publish test (only after Jason approves)
5. Image quality iteration (after first publish confirms pipeline works)

## Decisions made autonomously

1. **Backend directory structure**: Spec says `src/broadcast/` but existing backend has no `src/` dir (code lives at root: routes/, services/, etc.). Creating `src/broadcast/` as specified since it's additive and keeps Broadcast cleanly isolated. The broadcast routes will be mounted in index.js — this is the minimal necessary modification to non-Broadcast code.
2. **Broadcast env var naming**: Using `BROADCAST_` prefix for new env vars (BROADCAST_WP_URL, BROADCAST_WP_USERNAME, BROADCAST_WP_APP_PASSWORD, BROADCAST_MAILCHIMP_API_KEY, etc.) to avoid collisions with existing Catapultor env vars.
3. **Auth reuse**: Broadcast will use the same Supabase Auth + requireAuth middleware as Catapultor. The broadcast_workspaces.owner_user_id will FK to auth.users.id (same as stores.user_id).
4. **Package format**: Using CommonJS (require/module.exports) to match existing backend codebase, not ESM.
5. **OpenAI gpt-image-1 failure**: Primary OpenAI image gen failed during test (model may require specific access). Ideogram V2 fallback succeeded. Image quality is decent but not as good as manual ChatGPT — will need iteration.
6. **Image persistence**: Changed saveAsset to always download external URLs and re-upload to Supabase Storage. Ideogram returns ephemeral URLs that expire in hours. Fixed for test article retroactively.
7. **Env var naming fallback**: Adapters now try multiple env var name patterns (BROADCAST_WP_*, WP_*, WORDPRESS_*, etc.) since Jason may use different naming than the spec assumed.

## Blockers requiring Jason

1. **Railway env vars not detected** — Jason reported all 6 credentials set, but `/api/broadcast/health/credentials` shows all as missing. The `envHints` diagnostic returned empty (`[]`), meaning no env vars matching any of the tried patterns exist on the Railway service. **Jason needs to verify:** (a) the vars are on the correct Railway service (catapultor-backend, not a different one), (b) the var names. The code tries these patterns:
   - WordPress: `BROADCAST_WP_URL` / `WP_URL` / `WORDPRESS_URL`, `BROADCAST_WP_USERNAME` / `WP_USERNAME` / `WORDPRESS_USERNAME`, `BROADCAST_WP_APP_PASSWORD` / `WP_APP_PASSWORD` / `WORDPRESS_APP_PASSWORD`
   - Facebook: `BROADCAST_META_PAGE_ACCESS_TOKEN` / `META_PAGE_ACCESS_TOKEN` / `FB_PAGE_ACCESS_TOKEN`, `BROADCAST_META_PAGE_ID` / `META_PAGE_ID` / `FB_PAGE_ID`
   - Mailchimp: `BROADCAST_MAILCHIMP_API_KEY` / `MAILCHIMP_API_KEY`, `BROADCAST_MAILCHIMP_AUDIENCE_ID` / `MAILCHIMP_AUDIENCE_ID`

**Diagnostic URL:** https://catapultor-backend-production-c071.up.railway.app/api/broadcast/health/credentials

**NOTE:** All generation (headlines, images, article, atoms, variants) works fine. Only external publishing is blocked.

Known at kickoff (non-blocking):
- Voice profile seed content — working around by seeding placeholder voice profile
- Proven image prompt patterns — working around by building pattern library structure empty, iterating with best-guess prompts
- Platform API access for Tier 2/3 (TikTok, Pinterest, X tier, YouTube quota) — does NOT block Phase 1a

**Jason manual action items (not code):**
- **Pinterest privacy policy URL:** Update Pinterest developer app (ID 1562567) at developers.pinterest.com → Manage → app settings. Change privacy policy URL from temporary GitHub URL to `https://www.catapultor.com/privacy`. Same update needed for any other platform dev portal that was submitted with a temporary URL.
- **Meta app review:** Verify Meta app permissions include Threads + IG Content Publishing for future phases.

## Phase 1a acceptance criteria

- [x] All Broadcast Supabase tables live with correct schema and indexes
- [x] Fried Ocean seeded as workspace 1 with voice profile + placeholder voice examples + initial prompt library
- [x] Article workflow wizard functional end-to-end: idea -> headline (iterable) -> image (iterable) -> article -> review
- [ ] WordPress publish flow working (article lands on friedocean.com) — **pending Jason credential verification**
- [ ] Facebook publish flow working (post lands on FO FB page) — **pending Jason credential verification**
- [ ] Mailchimp newsletter send flow working (campaign created and sent) — **pending Jason credential verification**
- [x] Basic schedule grid functional
- [x] Meta token refresh cron running — daily cron checks env-var and DB-stored tokens, alerts on expiry
- [x] Dashboard shell + prompts library + voice editor screens shipped
- [ ] Image service produces hero images matching Jason's manual ChatGPT output quality bar — **pending Jason review** (Ideogram fallback used; OpenAI gpt-image-1 failed)
- [ ] End-to-end smoke test passed — **pending Jason review of generated content + credential verification**
- [x] All tests passing (28 state machine + 5 prompt service integration)
- [x] Backend deployed to Railway
- [x] Frontend deployed via Vercel
- [x] Deployed URLs documented in this file

## Deployed URLs

- Backend API: https://catapultor-backend-production-c071.up.railway.app/api/broadcast/health (confirmed OK 2026-04-16)
- Frontend app: https://www.catapultor.com/broadcast (deployed via Vercel 2026-04-16)
- Admin entry point: https://www.catapultor.com/broadcast (behind Catapultor auth)

## Generation test evidence (2026-04-16 session 2 — no publishing)

- **Test article ID:** 52bb5195-7374-46df-b21c-61fac8cf364d
- **Idea seed:** "A mid-level executive is named Employee of the Month for the 47th consecutive time after it's quietly revealed he is the only employee."
- **Dashboard URL:** https://www.catapultor.com/broadcast/articles/52bb5195-7374-46df-b21c-61fac8cf364d
- **Final state:** atomized

### Pipeline results
| Step | Result | Cost |
|---|---|---|
| Headlines | 10 generated, first auto-selected: "Local Executive Wins Employee of the Month for 47th Time in a Row" | 79c |
| Hero Image | Ideogram V2 (OpenAI gpt-image-1 failed), persisted to Supabase Storage | 87c |
| Article Body | 4075 chars, full satirical article with fake expert quotes | 90c |
| SEO Metadata | Title, description, 5 keywords generated | 30c |
| Atomization | 12 atoms (headlines, punchlines, quotes, stats, absurd details) | 91c |
| Facebook Variant | Caption + hashtags generated | 22c |
| Newsletter Variant | Blurb + CTA generated | 23c |
| **Total** | | **~$4.22** |

### Hero image (regenerated with OpenAI gpt-image-1 + quality gate — session 3)
- **URL:** https://ilrmcnifkdpowsoryiam.supabase.co/storage/v1/object/public/post-images/broadcast/articles/52bb5195-7374-46df-b21c-61fac8cf364d/hero_image_1776362779086.png
- **Provider:** OpenAI gpt-image-1 (primary, no fallback needed)
- **Quality gate:** PASSED on first attempt. GPT-4o vision check confirmed: photorealistic, no text violations, no anatomy issues, no unexplained objects.
- **Cost:** 96 cents (brief: 46c, prompt build: 39c, generation: 8c, quality check: 3c)
- **Old Ideogram image** (for comparison): https://ilrmcnifkdpowsoryiam.supabase.co/storage/v1/object/public/post-images/broadcast/articles/52bb5195-7374-46df-b21c-61fac8cf364d/hero_image_persisted.png

### Credential verification (no publishing attempted)
- WordPress: NOT VERIFIED (env vars not detected on Railway)
- Facebook: NOT VERIFIED (env vars not detected on Railway)
- Mailchimp: NOT VERIFIED (env vars not detected on Railway)
- OpenAI: VERIFIED (generation succeeded)
- Supabase: VERIFIED (all reads/writes succeeded)

## Smoke test evidence (external publishing)

(Pending — requires Railway credential verification + Jason approval of generated content)
- Published WordPress URL: (pending)
- Published Facebook post URL: (pending)
- Mailchimp campaign URL: (pending)

## Phase 1a -> Phase 1b transition

Do NOT start Phase 1b until Phase 1a is fully deployed and smoke-tested. When Phase 1a is complete, report to Jason with acceptance criteria check, deployed URLs, and smoke test evidence.
