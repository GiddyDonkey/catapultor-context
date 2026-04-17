# catapultor-context

**Last synced:** 2026-04-16

Public mirror of non-sensitive context files from the private `catapultor-backend` repo. These files provide project context (backlog, specs, status, voice examples) that can be fetched by Claude chat sessions via `web_fetch` since Claude chat cannot access private repos.

## What's here

| File | Purpose |
|---|---|
| `CATAPULTOR_BACKLOG.md` | Running backlog for Catapultor + Broadcast module |
| `BROADCAST_SPEC.md` | Full build specification for the Broadcast module |
| `BROADCAST_STATUS.md` | Current build status, progress, blockers, deployed URLs |
| `BROADCAST_CLAUDE.md` | Operating rules for Claude Code sessions on Broadcast |
| `broadcast-seeds/README.md` | Guide to the seed data files |
| `broadcast-seeds/voice-examples.md` | Gold-tier voice examples for Fried Ocean |

## What's NOT here (private)

- `CLAUDE.md` — may reference credentials and infrastructure details
- `broadcast-seeds/existing-prompts.md` — production-tested prompts (competitive advantage)
- `broadcast-seeds/image-prompt-patterns.md` — image generation patterns (competitive advantage)
- Any source code, env vars, tokens, or credentials

## Fetching in Claude chat

Use these raw URLs in Claude chat with `web_fetch`:

```
https://raw.githubusercontent.com/GiddyDonkey/catapultor-context/main/BROADCAST_SPEC.md
https://raw.githubusercontent.com/GiddyDonkey/catapultor-context/main/BROADCAST_STATUS.md
https://raw.githubusercontent.com/GiddyDonkey/catapultor-context/main/CATAPULTOR_BACKLOG.md
https://raw.githubusercontent.com/GiddyDonkey/catapultor-context/main/BROADCAST_CLAUDE.md
```

## Sync

These files are synced from `catapultor-backend` using `scripts/sync-public-context.sh` in the backend repo. Run it after updating any of the mirrored files.

## Owner

Jason / JBW Investments — Catapultor project
