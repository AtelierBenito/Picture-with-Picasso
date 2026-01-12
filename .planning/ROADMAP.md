# Roadmap: Picture with Picasso

## Overview

Transform the silent MVP video generator into an immersive, interactive experience where Picasso speaks personalized dialog, recognizes the owner (BENIT), and delivers professionally compressed videos with branded messaging. The journey moves from input handling through AI-powered audio generation, lip sync, and polished delivery.

## Domain Expertise

None (n8n workflow automation with external API integrations)

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [ ] **Phase 1: Input Processing** - Parse user input, resolve interactions, route to pipelines
- [ ] **Phase 2: Content Moderation** - OpenAI Moderation API for input validation
- [ ] **Phase 3: Face Recognition** - Azure Face API setup, BENIT identification
- [ ] **Phase 4: Script Generation** - Grok AI via OpenRouter, Picasso character dialog
- [ ] **Phase 5: Voice Configuration** - ElevenLabs voices (Picasso, BENIT clone)
- [ ] **Phase 6: Audio Production** - TTS generation, ambient sounds, mixing
- [ ] **Phase 7: Lip Sync Integration** - LatentSync via fal.ai
- [ ] **Phase 8: Compression & Delivery** - Cloudinary, binary files, Telegram fix
- [ ] **Phase 9: Enhanced Delivery** - Branded messages, hashtags, formatting
- [ ] **Phase 10: Integration & Polish** - End-to-end testing, error handling

## Phase Details

### Phase 1: Input Processing
**Goal**: Parse `/interaction:` directive and `message` field, resolve interaction for video prompt, route data to appropriate pipelines
**Depends on**: Nothing (builds on existing workflow)
**Research**: Unlikely (internal n8n patterns)
**Plans**: TBD

Key work:
- Parse Telegram caption for `/interaction:` directive
- Parse web form fields (interaction, message, name)
- Resolve interaction (user-provided → enhance, empty → random from curated list)
- Inject `resolvedInteraction` into Kling O1 prompt

### Phase 2: Content Moderation
**Goal**: Validate user input against inappropriate content before processing
**Depends on**: Phase 1 (needs parsed input)
**Research**: Likely (external API)
**Research topics**: OpenAI Moderation API endpoints, request format, category handling
**Plans**: TBD

Key work:
- OpenAI Moderation API integration (free tier)
- Check `message` field for flagged content
- Check `interaction` field for flagged content
- Rejection flow with friendly user message

### Phase 3: Face Recognition
**Goal**: Detect faces in uploaded photo and identify BENIT for personalized experience
**Depends on**: Phase 1 (needs photo input)
**Research**: Likely (external API setup)
**Research topics**: Azure Face API free tier, PersonGroup creation, face training, detect + identify flow
**Plans**: TBD

Key work:
- Azure Face API account setup (free tier: 30K/month)
- Create PersonGroup "picasso-visitors"
- Train BENIT reference face
- Face detection node in workflow
- Face identification node (BENIT vs unknown)
- Output: `isBenit`, `faceCount`, `faces[]` with gender/age

### Phase 4: Script Generation
**Goal**: Generate dynamic Picasso dialog based on visitor context
**Depends on**: Phase 2 (moderated input), Phase 3 (face recognition)
**Research**: Likely (external API)
**Research topics**: OpenRouter API, Grok model (x-ai/grok-4.1-fast), prompt engineering for character
**Plans**: TBD

Key work:
- OpenRouter API integration
- Grok model configuration
- Picasso character system prompt (personality, Spanish phrases, content rules)
- User prompt template (isBenit, faceCount, userName, userMessage)
- Output moderation (check generated script)
- Regeneration flow if output flagged

### Phase 5: Voice Configuration
**Goal**: Set up voices for all speakers (Picasso, BENIT, visitors)
**Depends on**: Nothing (can parallel with Phase 4)
**Research**: Likely (external API)
**Research topics**: ElevenLabs Voice Design API, Voice Cloning API, voice IDs, settings
**Plans**: TBD

Key work:
- ElevenLabs direct API setup (not via fal.ai)
- Create Picasso voice (Voice Design - Spanish accent, 74-year-old, warm)
- Create BENIT voice clone (Professional Voice Cloning)
- Select/create visitor voices (male, female)
- Voice assignment logic based on face recognition

### Phase 6: Audio Production
**Goal**: Generate all audio tracks and mix into final audio
**Depends on**: Phase 4 (script), Phase 5 (voices)
**Research**: Likely (external API)
**Research topics**: ElevenLabs TTS API, Sound Effects API, audio mixing in n8n
**Plans**: TBD

Key work:
- Picasso TTS generation (main dialog)
- BENIT TTS generation (when detected)
- Visitor sound effects (joy sounds: "Oh! Wow!")
- Studio ambience generation (Sound Effects API)
- Audio mixing (dialog foreground, ambient -20dB background)

### Phase 7: Lip Sync Integration
**Goal**: Synchronize audio with video using LatentSync
**Depends on**: Phase 6 (audio), existing Kling O1 video
**Research**: Likely (external API)
**Research topics**: LatentSync via fal.ai, submit/poll pattern, input/output formats
**Plans**: TBD

Key work:
- LatentSync submit node (video URL + audio)
- Status polling loop (similar to Kling pattern)
- Result retrieval
- Error handling for sync failures

### Phase 8: Compression & Delivery
**Goal**: Compress video for efficient delivery and fix Telegram display issues
**Depends on**: Phase 7 (lip-synced video)
**Research**: Likely (external API)
**Research topics**: Cloudinary video API, compression settings, n8n Cloudinary node
**Plans**: TBD

Key work:
- Cloudinary account/credentials setup
- Video upload with compression (q_auto:eco, f_auto:video, c_limit,w_1080)
- Reduce file size from 20-30MB to 5-10MB
- Binary file delivery (actual files, not URLs)
- Dual filename convention (internal archive + user-facing)
- Telegram display fix (explicit width/height)

### Phase 9: Enhanced Delivery
**Goal**: Add branding, hashtags, and professional formatting to delivery messages
**Depends on**: Phase 8 (delivery pipeline)
**Research**: Unlikely (text formatting only)
**Plans**: TBD

Key work:
- Branded message template with attribution
- Hashtag-ready captions for social sharing
- "An Atelier Benito joint, produced by RobotBird.AI"
- Format for Telegram and email

### Phase 10: Integration & Polish
**Goal**: End-to-end testing, error handling, edge cases
**Depends on**: All previous phases
**Research**: Unlikely (internal testing)
**Plans**: TBD

Key work:
- Full pipeline integration testing
- Error handling at each stage
- Graceful fallbacks (e.g., silent video if audio fails)
- Edge cases (no face detected, multiple people, etc.)
- Performance optimization

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Input Processing | 0/TBD | Not started | - |
| 2. Content Moderation | 0/TBD | Not started | - |
| 3. Face Recognition | 0/TBD | Not started | - |
| 4. Script Generation | 0/TBD | Not started | - |
| 5. Voice Configuration | 0/TBD | Not started | - |
| 6. Audio Production | 0/TBD | Not started | - |
| 7. Lip Sync Integration | 0/TBD | Not started | - |
| 8. Compression & Delivery | 0/TBD | Not started | - |
| 9. Enhanced Delivery | 0/TBD | Not started | - |
| 10. Integration & Polish | 0/TBD | Not started | - |
