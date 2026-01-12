# Picture with Picasso

## What This Is

An AI-powered video generation experience that creates personalized "meeting Picasso" moments. Users submit a photo via Telegram or web form and receive a video of themselves interacting with Pablo Picasso in his 1955 studio—with the upcoming addition of voice, face recognition, and dynamic AI-generated dialogue that responds to user input.

**Brand:** Atelier Benito
**Producer:** RobotBird.AI
**Attribution:** "An Atelier Benito joint, produced by RobotBird.AI"

## Core Value

**Visual quality and identity preservation.** The magic breaks when:
- Figures interact unnaturally (quirky/stiff movements)
- People don't look like themselves (face morphing, stretched features)
- Picasso isn't proportionally scaled (he's 5'4" / 163cm)
- Overly expressive AI faces destroy likeness

If users don't recognize themselves, everything else fails.

## Requirements

### Validated

- ✓ Kling O1 video generation via fal.ai — Phase 1
- ✓ Telegram bot input channel — Phase 1
- ✓ Web form input channel — Phase 1
- ✓ Google Drive archival — Phase 1
- ✓ Shuffle bag Picasso image selection — Phase 1
- ✓ Reference-to-video with elements — Phase 1

### Active

**Phase 2: Interactive Audio Pipeline**
- [ ] User input parsing (`/interaction:` for video, `message` for audio)
- [ ] Content moderation (OpenAI Moderation API - free)
- [ ] Face recognition (Azure Face API - identify BENIT vs visitors)
- [ ] AI script generation (Grok via OpenRouter)
- [ ] Voice assignment (BENIT clone, Picasso voice, visitor sounds)
- [ ] ElevenLabs TTS for Picasso dialog (direct API)
- [ ] ElevenLabs Sound Effects for studio ambience (direct API)
- [ ] LatentSync lip sync integration (via fal.ai)
- [ ] Audio mixing (dialog + ambient)

**Phase 3: Compression & Delivery**
- [ ] Cloudinary video compression (reduce 20-30MB → 5-10MB)
- [ ] Binary file delivery (actual files, not URLs)
- [ ] Dual filename convention (internal archive + user-facing)
- [ ] Telegram display fix (explicit dimensions)

**Phase 4: Enhanced Delivery**
- [ ] Branded message templates
- [ ] Hashtag-ready captions
- [ ] Professional formatting for social sharing

### Out of Scope

- Mobile apps (iOS/Android) — Future milestone (Phase 8)
- Full SaaS with Supabase/Stripe — Future milestone (Phase 7)
- Web frontend on RobotBird.io — Future milestone (Phase 6)
- Social posting buttons/integration — Future milestone (Phase 5)
- Video watermarking — Future milestone (Phase 5)
- Simple authentication — Future milestone (Phase 5-7)
- WhatsApp integration — Future milestone (Phase 8)
- Email input/output channel — Not planned

## Context

**Current State:** Phase 1 Complete (Core MVP with silent video)

**Input Channels:** Telegram Bot + Web Form

**Existing Workflow:** `source/workflows/Picture with Picasso -v.3-v2.4.json`
- Active on n8n (workflow ID: Q2Z6nJYPotQnhlwj)
- Uses Kling O1 reference-to-video via fal.ai
- "Group Reunion" prompt with @Element1 (visitors) and @Element2 (Picasso)
- 9:16 portrait, 10 seconds, audio generation enabled (but no lip sync yet)

**Design Documents:**
- User Experience Design v4.0: `docs/design/docs-design-Picture_with_Picasso_User_Experience-v4.0-2026_01_11.md`
- Project Roadmap v1.0: `docs/design/docs-design-Project_Roadmap-v1.0-2026_01_10.md`

**Current Cost Per Video:** ~$1.00 (Kling O1 only)
**Target Cost (Phase 4):** ~$1.26 (+ TTS, lip sync, compression)

**Project Structure:**
- GSD management: `.planning/` (PROJECT.md, ROADMAP.md, STATE.md, phases/)
- Artifacts: `source/workflows/`, `docs/design/`, `versions/workflows/`
- Session files: `AA-*.md` at project root

## Constraints

- **Platform:** n8n workflow automation (existing infrastructure)
- **Video API:** fal.ai for Kling O1 and LatentSync
- **Audio API:** ElevenLabs direct API (TTS + Sound Effects)
- **Free Tier Services:** OpenAI Moderation (free), Azure Face API (30K/month free)
- **Budget Consciousness:** Prefer free/low-cost services where possible
- **Visual Quality:** Identity preservation is non-negotiable

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Kling O1 over Wan 2.5 | Better character reference handling, native audio | ✓ Good |
| ElevenLabs direct API | Full control, not limited by fal.ai wrapper | — Pending |
| fal.ai for video only | Kling O1 + LatentSync; audio handled separately | — Pending |
| Azure Face API for recognition | Free tier (30K/month), reliable, BENIT identification | — Pending |
| Grok via OpenRouter for scripts | Creative, witty, character roleplay strength | — Pending |
| OpenAI Moderation (free) | Zero-cost content safety | — Pending |
| Telegram + Web Form only | No email channel needed | — Pending |
| Phases 2-4 before social posting | Ship working audio before viral features | — Pending |

---
*Last updated: 2026-01-12 after GSD initialization*
