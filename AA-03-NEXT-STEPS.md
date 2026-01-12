# Next Steps - Prioritized Action Items

**Last Updated:** 2026-01-11
**Current Phase:** Prompt Testing + Pre-Build Setup
**Next Phase:** Phase 2 Build (Full User Experience)

---

## Immediate: Test Prompt v1.9.2 (15 min)

**NEW prompts created - test before Phase 2 build:**

### Test Instructions
1. Copy prompt from `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.9.2-2026_01_11.md`
2. Copy negative prompt from `source/prompts/source-prompts-Kling_O1_Negative_Prompt-v1.9.2-2026_01_11.md`
3. Paste into `Prepare Kling Elements1` node in n8n
4. Generate 2-3 test videos

### Verification Checklist
- [ ] All figures interact with EACH OTHER (not just Picasso)
- [ ] Physical contact present (hugs, embraces, arm around shoulder)
- [ ] No isolated/observer figures standing apart
- [ ] Visitor faces match source exactly (no drift)
- [ ] Picasso matches source exactly
- [ ] Energy is ecstatic reunion, not formal greeting

---

## After Testing: Pre-Build Setup (45-60 min)

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
- [ ] Add input parsing node (Telegram caption, form fields)
- [ ] Add user-directed interaction resolver
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

- [x] Prompt v1.9.2 - Group reunion dynamics (2026-01-11)
- [x] User-directed interaction feature design (2026-01-11)
- [x] User Experience v4.0 design doc (2026-01-11)
- [x] Interactive Audio Pipeline architecture design (2026-01-10)
- [x] Lip sync analysis (LatentSync is audio-driven) (2026-01-10)
- [x] Content moderation strategy (OpenAI - FREE) (2026-01-10)
- [x] Face recognition cost analysis (Azure - FREE tier) (2026-01-10)
- [x] AI model selection (Grok 4.1 Fast via OpenRouter) (2026-01-10)
- [x] Phase 2 Setup Guide created (2026-01-10)
- [x] Kling O1 audio limitation confirmed (2026-01-09)
- [x] Prompt character limit fix v1.5 (2026-01-09)

---

## Key Documents

| Document | Purpose |
|----------|---------|
| `docs/design/docs-design-Picture_with_Picasso_User_Experience-v4.0-2026_01_11.md` | **MAIN DESIGN** |
| `docs/guides/docs-guides-Phase_2_Setup_Guide-v1.0-2026_01_10.md` | Setup checklist |
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.9.2-2026_01_11.md` | Latest prompt |

---

## Ready Check

When testing and setup complete, message: **"Ready to build Phase 2!"**

We'll start with Phase 2A (Core Audio) and progress through each phase.

---

*Last updated: 2026-01-11*
