# BROADCAST_CLAUDE.md — Operating rules for Claude Code on the Broadcast module

Read this at the start of every session before touching any Broadcast code.

## Operating mode: AUTONOMOUS

Jason does not want to be interrupted. Work until Phase 1a is complete and deployed. Do not stop to ask for permission. Do not ask clarifying questions. Do not pause between steps waiting for approval.

**When you hit ambiguity:** pick the most reasonable default, log your decision in `BROADCAST_STATUS.md` under "Decisions made autonomously," and keep moving.

**When you hit an error:** fix it yourself. Loop on test-and-fix until passing. Never leave broken code. Never hand back partial work.

**When you finish a logical unit:** commit with `[Broadcast]` prefix and a descriptive message, push to remote, then immediately continue to the next unit.

**The only acceptable stopping points:**
- Phase 1a acceptance criteria all met and system is deployed and smoke-tested end-to-end
- A hard external blocker you cannot resolve (third-party API credential Jason has not provided and cannot be defaulted)

If you hit a hard blocker, document it clearly in `BROADCAST_STATUS.md` under "Blockers requiring Jason" — but before stopping, confirm you've actually exhausted your own options (checked env vars, read existing Catapultor integrations for reusable credentials, tried reasonable fallbacks).

## Session startup checklist

1. Read `BROADCAST_SPEC.md` (full spec)
2. Read `BROADCAST_STATUS.md` (current phase, last session's progress)
3. Read root `CLAUDE.md` for project-wide Catapultor rules
4. Confirm Railway and Supabase are accessible (real environments, no mocks)
5. Begin work immediately on the next-up task from `BROADCAST_STATUS.md`

## Core rules

1. Additive only. Broadcast lives in `src/broadcast/` in both repos. Never modify non-Broadcast Catapultor code.
2. Use what exists. Check existing Catapultor and Jason's external stack (Spec Section 2) before building anything.
3. Namespace everything. Tables `broadcast_`, routes `/api/broadcast/` and `/broadcast/`.
4. Real infra. Real Railway, real Supabase. No mocks.
5. Commit discipline. Every 3-5 fixes, `[Broadcast]` prefix, push to remote.
6. Deploy continuously. Backend to Railway, frontend via `npx vercel --prod`.
7. Test continuously. Write tests alongside services. Run before commit. Fix failures immediately.
8. Loop until passing. Never leave broken code.
9. No credential changes. If missing, log blocker, continue with unblocked work.
10. Schema evolves freely during initial build. After first production publish, additive migrations only.
11. Image service is sacred. Quality bar: "matches Jason's manual ChatGPT output." Iterate until true before declaring Phase 1a done.
12. Use non-interactive flags on CLI. `--yes`, `-y`, `CI=1`, etc. Never hang on a prompt.

## Continuous testing requirements

- Unit tests: state machine transitions, prompt rendering with voice injection, platform adapter interfaces
- Integration tests: WordPress publish, Facebook publish, Mailchimp publish, OpenAI image gen, Ideogram fallback
- Run tests before every commit
- End of Phase 1a: full end-to-end smoke test creating real Fried Ocean article from idea seed through WP + FB + newsletter publish, verified each destination

## Session end checklist (when Phase 1a is complete)

Update `BROADCAST_STATUS.md` with:
- Phase 1a acceptance criteria — every item checked or explicitly noted as blocked
- Deployed URLs (Railway backend, Vercel frontend, admin routes)
- Smoke test results with evidence (WP URL, FB post URL, Mailchimp campaign URL)
- Decisions made autonomously (every non-obvious choice + rationale)
- Known follow-ups for Phase 1b
- Blockers that required Jason (only if you couldn't resolve)

## Quality bars

- Voice consistency: every text generation injects active voice profile + few-shot examples. No exceptions.
- Error visibility: no silent failures. Everything logged and surfaced per alerting spec.
- Non-destructive: existing Fried Ocean operations must not break. Make and Later continue in parallel.
- Image quality: matches Jason's manual ChatGPT output. Iterate until true.
- Deployed and working: Phase 1a is not done until a real article has been published end-to-end through the system.

## Public context mirror

A public repo at https://github.com/GiddyDonkey/catapultor-context mirrors non-sensitive context files (backlog, spec, status, voice examples) so Claude chat sessions can fetch them via `web_fetch`.

After updating CATAPULTOR_BACKLOG.md, BROADCAST_SPEC.md, BROADCAST_STATUS.md, BROADCAST_CLAUDE.md, or broadcast-seeds/voice-examples.md, run:

```bash
bash scripts/sync-public-context.sh
```

This copies the designated files, scrubs for secrets, commits, and pushes to the public repo. Do NOT manually copy files — always use the script.

## Persistent Agents

### Backlog Progression Agent

An autonomous GitHub Actions agent that continuously works through the Catapultor backlog on an 8-hour cadence.

- **Workflow:** `.github/workflows/backlog-progression-agent.yml`
- **Schedule:** Every 8 hours (cron `0 */8 * * *`) + manual dispatch
- **Tracking:** GitHub issue "Backlog Progression Agent — Activity Log" (label: `agent-log`)
- **Allowlist:** `AGENT_SAFE_ITEMS.md` — only items marked `agent_safe: yes` are eligible
- **Kill switch:** Repository variable `AGENTS_ENABLED` — set to `false` to stop. Script: `bash scripts/halt-agents.sh`
- **Schedule tuning:** `bash scripts/set-agent-schedule.sh "0 */6 * * *"`
- **Budget:** $3/run, cost ceiling = runs/day x $3
- **Branch rule:** Agent NEVER commits to main. All work on `agent/*` branches, reaches main via PR only.
- **Review:** Jason reviews `[Agent]`-prefixed PRs before merging. Check tracking issue for run summaries.

### Morning review checklist for Jason

1. Read tracking issue (latest comments)
2. Check open PRs with `[Agent]` prefix
3. Verify cost in Anthropic + OpenAI dashboards
4. Review and merge good PRs, close bad ones
5. Check if `AGENTS_ENABLED` is still `true`
