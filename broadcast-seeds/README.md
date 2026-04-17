# Broadcast Seed Content

This folder contains seed content for the Broadcast module. Claude Code should read all files in this folder during the Phase 1a prompt-library seeding step and use them to populate `broadcast_prompts` and `broadcast_voice_examples` as specified below.

## Files

- `existing-prompts.md` — Jason's production-tested prompts, used manually across ChatGPT and Claude today. These are the authoritative v1 content for the Broadcast prompt library. Map each prompt to the corresponding `broadcast_prompts.key` per the mapping table in that file. Do not replace with generic templates.

- `voice-examples.md` — Gold / silver / bronze tier examples of Fried Ocean content across categories. Load into `broadcast_voice_examples` for few-shot injection during generation. Jason is curating this over time; may be partial at any given moment. If empty or minimal, seed with placeholder content and continue — do not block on this.

- `image-prompt-patterns.md` — Verbatim manual image prompts that produced Jason's best hero images, plus observations on what works and what fails. Use these to seed the proven-pattern fragment library referenced in Spec Section 5.3 (the image service's prompt construction step). If empty or minimal, fall back to patterns in `existing-prompts.md` — do not block on this.

## Mapping: existing prompts → Broadcast prompt keys

| Existing prompt (in `existing-prompts.md`) | Broadcast `broadcast_prompts.key` | Notes |
|---|---|---|
| Image Generation System Prompt (Claude) | `image_brief_builder` | Use for generating structured JSON briefs from article context |
| Image Master Prompt (ChatGPT) | `image_prompt_builder_openai` | Use for rendering brief → OpenAI-optimized prompt string |
| Headline & Satire Prompt (ChatGPT) | `headline_ideation` | Generates 3-5 candidates for iteration loop |
| Headline & Satire Prompt (reduced) | `headline_refinement` | Single-headline refinement based on user feedback; derive from ideation prompt |
| Article Writer / Master Prompt (ChatGPT) | `article_body` | Takes locked headline, produces full article body |
| SEO Optimizer (ChatGPT) | `article_seo_meta` | Strict JSON output: focus keyphrase, meta description, excerpt |

## Re-seeding behavior

If these files are updated, Claude Code can re-run the seed step to refresh the prompt library. Use the versioning field in `broadcast_prompts` — bump version, set old version inactive. Never overwrite history.

## Priority order if content is missing

1. `existing-prompts.md` — Broadcast cannot run without these. Block until present or use minimal placeholders and flag to Jason.
2. `voice-examples.md` — Broadcast runs with placeholder examples if empty; voice quality will suffer but system functions.
3. `image-prompt-patterns.md` — Image service uses brief-builder patterns from `existing-prompts.md` if this file is empty.

## Implementation notes for Broadcast build session

**Two-stage image prompt architecture:**
The existing image prompts (one from Claude, one from ChatGPT) are NOT duplicates. They are complementary:
- The Claude system prompt emphasizes style rotation, character diversity enforcement, and moderation workarounds → use this as `image_brief_builder` (produces structured JSON brief)
- The ChatGPT Master Prompt emphasizes composition template and scene construction → use this as `image_prompt_builder_openai` (takes brief → renders provider-optimized final prompt string)

**Dynamic character diversity tracking:**
The image brief builder prompt references "Examples used so far" as a list of previously-used character descriptions. Broadcast should track this dynamically — on each image generation, record the character description used in article metadata (or a dedicated `broadcast_image_character_log` table), and inject the last 20-30 used characters into the image brief prompt context so repetition is avoided automatically.

**Article structure rotation:**
The article_body prompt requires rotating article formats (narrative scene / institutional report / observational breakdown / timeline / internal memo / slow-burn / fragmented / quote-heavy). Broadcast should pass "recent article structures used" into the article_body prompt context so recent patterns are avoided. Track structure used on each article.

**Strict JSON output for SEO:**
The SEO optimizer prompt has a strict output format (Focus Keyphrase / Meta Description / Excerpt). Update the prompt when seeding to explicitly request JSON output so parsing is deterministic. The semantic content stays the same.
