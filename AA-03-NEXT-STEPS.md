# Next Steps - Prioritized Action Items

**Last Updated:** 2026-01-11
**Current Phase:** Pre-Build Setup
**Next Phase:** Phase 2 Build (Interactive Audio Pipeline)

---

## Immediate: Pre-Build Setup (45-60 min)

**Complete before build session:**

### Priority 1: Voice Setup (20 min)
- [ ] Create ElevenLabs account
- [ ] Get ElevenLabs API key
- [ ] Create Picasso voice (Voice Design)
- [ ] Create your voice clone (Instant Clone)
- [ ] Save both Voice IDs

### Priority 2: Face Recognition (15 min)
- [ ] Create Azure account
- [ ] Create Face API resource (Free F0 tier)
- [ ] Get API key and endpoint
- [ ] Prepare 3-5 reference photos of yourself

### Priority 3: AI Services (10 min)
- [ ] Create OpenRouter account
- [ ] Get OpenRouter API key
- [ ] Create/verify OpenAI account
- [ ] Get OpenAI API key

### Priority 4: Materials (5 min)
- [ ] Save reference photos to `Picasso Images/BENIT_Reference/`
- [ ] Prepare 2-3 test visitor photos
- [ ] Fill out credential summary card

**Full Guide:** `docs/guides/docs-guides-Phase_2_Setup_Guide-v1.0-2026_01_10.md`

---

## Phase 2 Build Sessions (7-10 hours total)

### Phase 2A: Core Audio (2-3 hours)
- [ ] Add Picasso TTS node (ElevenLabs Direct API)
- [ ] Add ambient sound generation node
- [ ] Add audio mixing node
- [ ] Add LatentSync submit node
- [ ] Add LatentSync polling loop
- [ ] Test end-to-end with static script

### Phase 2B: Face Recognition (2 hours)
- [ ] Create PersonGroup in Azure
- [ ] Upload and train with your reference photos
- [ ] Add face detection node
- [ ] Add face identification node
- [ ] Add voice assignment logic
- [ ] Test BENIT recognition

### Phase 2C: Interactive Input (2-3 hours)
- [ ] Add input content moderation node
- [ ] Add Grok script generation node
- [ ] Add output content moderation node
- [ ] Connect user message to script generation
- [ ] Test full interactive flow

### Phase 2D: Polish (1-2 hours)
- [ ] Add visitor joy sounds
- [ ] Refine audio mixing levels
- [ ] Add error handling
- [ ] Full integration testing
- [ ] Update documentation

---

## Completed (Archive Reference)

- [x] Interactive Audio Pipeline architecture design (2026-01-10)
- [x] Lip sync analysis (LatentSync is audio-driven) (2026-01-10)
- [x] Content moderation strategy (OpenAI - FREE) (2026-01-10)
- [x] Face recognition cost analysis (Azure - FREE tier) (2026-01-10)
- [x] AI model selection (Grok 4.1 Fast via OpenRouter) (2026-01-10)
- [x] ElevenLabs Integration guide (2026-01-10)
- [x] Phase 2 Setup Guide created (2026-01-10)
- [x] Audio pipeline architecture research (2026-01-10)
- [x] Webhook vs polling analysis (2026-01-10)
- [x] Kling O1 audio limitation confirmed (2026-01-09)
- [x] Prompt character limit fix v1.5 (2026-01-09)

---

## Key Documents

| Document | Purpose |
|----------|---------|
| `docs/guides/docs-guides-Phase_2_Setup_Guide-v1.0-2026_01_10.md` | **START HERE** - Setup checklist |
| `docs/design/docs-design-Interactive_Audio_Pipeline-v1.0-2026_01_10.md` | Full architecture |
| `docs/design/docs-design-ElevenLabs_Integration-v1.0-2026_01_10.md` | Voice setup |
| `docs/analysis/docs-analysis-Lip_Sync_Audio_Pipeline-v1.0-2026_01_10.md` | Lip sync details |

---

## Future Phases (Not Yet Scheduled)

### Phase 3: Compression & Delivery
- Video compression optimization
- Delivery channel integration
- Analytics tracking

### Phase 4: Advanced Features
- Multi-turn conversations
- Real-time avatar (different tech)
- Voice library expansion

---

## Ready Check

When setup is complete, message: **"Ready to build Phase 2!"**

We'll start with Phase 2A (Core Audio) and progress through each phase.

---

*Last updated: 2026-01-10*
