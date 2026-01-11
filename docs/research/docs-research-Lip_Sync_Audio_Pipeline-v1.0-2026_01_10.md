# Lip Sync Audio Pipeline Research

**Version:** v1.0-2026-01-10
**Purpose:** Research feasibility of adding audio to Kling O1 silent videos via post-processing

---

## Executive Summary

Kling O1's `reference-to-video` endpoint does NOT support `generate_audio`. This research evaluates post-processing solutions to add synchronized audio to the silent video output.

**Verdict:** HIGHLY FEASIBLE using fal.ai's integrated TTS and lip sync services.

---

## Problem Statement

### Current Situation
- Kling O1 generates 10-second videos with visible lip movements
- The `generate_audio` parameter is silently ignored by the API
- Videos are silent despite prompt instructions for audio
- Audio capability only exists on Kling 2.6 (which lacks elements array for identity)

### Desired Outcome
- Final video includes synchronized audio
- Picasso speaks with Spanish-accented voice
- Visitors have natural responses/laughter
- Ambient sounds match the period environment

---

## Research Approach

### Methods Used
1. Perplexity deep research (API unavailable - used web search)
2. Sequential thinking for structured analysis
3. Web search for lip sync APIs and services
4. fal.ai documentation scraping
5. n8n workflow template search

### Key Questions Investigated
1. Do lip-reading AI services exist that generate speech from silent video?
2. What APIs are available for adding audio to AI-generated videos?
3. Are services available on fal.ai (same platform as Kling)?
4. Can text-to-speech be synchronized to existing lip movements?
5. What n8n integrations exist?

---

## Findings: Available Services

### 1. Lip Sync Services on fal.ai

| Service | Endpoint | Cost | Features |
|---------|----------|------|----------|
| **LatentSync** | `fal-ai/latentsync` | $0.20/≤40s, $0.005/s after | ByteDance model, high quality |
| **Sync Lipsync v2.0** | `fal-ai/sync-lipsync/v2` | $0.70/min | Zero-shot, preserves speaker style |
| **Sync Lipsync Pro** | `fal-ai/sync-lipsync/v2/pro` | Premium tier | Ultra-realistic, 4K support |
| **MuseTalk** | `fal-ai/musetalk` | TBD | Real-time, websocket support |
| **Kling LipSync** | `fal-ai/kling-video/lipsync` | TBD | Native to Kling ecosystem |
| **VEED Lipsync** | `veed/lipsync` | $0.40/min | Simple integration |

**Recommended:** LatentSync - best price/quality ratio for our use case.

### 2. Text-to-Speech on fal.ai (ElevenLabs Integration)

| Model | Endpoint | Features |
|-------|----------|----------|
| **Eleven-v3** | `fal-ai/elevenlabs/tts/eleven-v3` | Latest quality |
| **Multilingual v2** | `fal-ai/elevenlabs/tts/multilingual-v2` | 29 languages, accent accuracy |
| **Turbo v2.5** | `fal-ai/elevenlabs/tts/turbo-v2.5` | 32 languages, lowest latency |

**Recommended:** Multilingual v2 - best for Spanish-accented Picasso voice.

### 3. External Lip Sync Services

| Service | Description | Integration |
|---------|-------------|-------------|
| **Sync Labs (sync.so)** | Industry leader, Y Combinator backed | Direct API, also on fal.ai |
| **Pika + ElevenLabs** | AI video platform with lip sync | Consumer-focused |
| **AKOOL** | Multilingual lip sync | Enterprise API |
| **HeyGen** | AI avatars with lip sync | Video platform |
| **Gooey.AI** | Workflow-based lip sync | Integrates with ElevenLabs |

---

## Findings: LatentSync API Details

### Input Schema
```json
{
  "video_url": "string (required) - URL of video to lip sync",
  "audio_url": "string (required) - URL of audio to sync",
  "guidance_scale": "number (optional) - Range 1-2, default 1.0",
  "seed": "integer (optional) - Random seed",
  "loop_mode": "enum (optional) - 'pingpong' or 'loop'"
}
```

### Supported Formats
- **Video:** MP4, MOV, WebM, M4V, GIF
- **Audio:** MP3, OGG, WAV, M4A, AAC

### Pricing
- Videos ≤40 seconds: $0.20 flat
- Longer videos: $0.005 per second

### Limitations
- Queue timeout: 3600 seconds (1 hour)
- No explicit max video duration
- Processing time varies with video length

---

## Findings: ElevenLabs TTS API Details

### Input Schema (Multilingual v2)
```json
{
  "text": "string (required) - Text to convert to speech",
  "voice_id": "string (required) - Voice identifier",
  "model_id": "string (optional) - Model to use",
  "speed": "number (optional) - 0.7-1.2, affects tempo"
}
```

### Features
- 29 languages supported
- Accent accuracy for Spanish
- Voice cloning capability
- Streaming support

---

## Findings: n8n Integration

### Existing Templates Found

1. **Create Talking Avatar Videos with ElevenLabs & Infinitalk**
   - URL: https://n8n.io/workflows/8378
   - Uses fal.ai for lip sync
   - Proven pattern for HTTP Request → fal.ai

2. **Create UGC Video Ads with GPT-4o, ElevenLabs & WaveSpeed**
   - URL: https://n8n.io/workflows/10070
   - Complete TTS + lip sync pipeline

3. **Generate Brand Marketing Shorts with GPT-4, FAL AI & ElevenLabs**
   - URL: https://n8n.io/workflows/5596
   - Includes Kling video generation

### Credential Setup
- Create HTTP Header Auth credential in n8n
- Header Name: `Authorization`
- Value: `Key <FAL_API_KEY>`

---

## Analysis: Two User-Proposed Approaches

### Approach A: Lip-Reading to Generate Speech
**Concept:** Analyze lip movements in silent video → AI determines what's being said → Generate matching audio

**Feasibility:** LIMITED
- True lip-reading AI exists for accessibility/transcription
- NOT designed to generate speech from visual lip movements
- Would require: lip-reading model + inference + TTS
- No integrated solution found

**Conclusion:** Not practical with current technology.

### Approach B: Define Script → TTS → Lip Sync
**Concept:** Pre-define dialog script → Generate audio via TTS → Sync audio to video lip movements

**Feasibility:** HIGH
- Mature technology stack
- All components available on fal.ai
- Proven n8n integration patterns
- Cost-effective (~$0.25 additional per video)

**Conclusion:** RECOMMENDED approach.

---

## Analysis: Why Lip Sync Works

The lip sync approach works because:

1. **Lip movements are generic** - Kling generates plausible mouth movements for "talking" but not specific phonemes
2. **AI lip sync is forgiving** - LatentSync/Sync Labs adjust lip movements to match audio
3. **Short duration** - 10 seconds is ideal for lip sync algorithms
4. **Single speaker focus** - Picasso is primary speaker, simpler sync

### Potential Quality Issues
- If Kling generates closed-mouth frames → lip sync can't create speech
- If multiple people talk simultaneously → harder to sync
- Audio/visual mismatch if script doesn't match apparent emotion

### Mitigation
- Keep script short and expressive
- Focus on Picasso as primary speaker
- Use exclamations and laughter (less precise sync needed)

---

## Cost Analysis

### Per-Video Cost Breakdown

| Component | Cost | Notes |
|-----------|------|-------|
| Kling O1 Reference-to-Video | $1.00 | 10s video |
| ElevenLabs TTS | ~$0.05 | Estimate for 10s audio |
| LatentSync | $0.20 | Flat rate for ≤40s |
| **TOTAL** | **~$1.25** | |

### Cost Comparison

| Approach | Cost | Audio Quality |
|----------|------|---------------|
| Kling O1 only (current) | $1.00 | None |
| Kling O1 + Audio Pipeline | $1.25 | Good |
| Kling 2.6 (no identity) | ~$1.40 | Native |

**Value Assessment:** 25% cost increase for significant quality improvement.

---

## Technical Feasibility Assessment

| Factor | Rating | Notes |
|--------|--------|-------|
| API Availability | ✅ High | All services on fal.ai |
| Integration Complexity | ✅ Medium | Similar to existing Kling pattern |
| Cost | ✅ Good | ~25% increase |
| Quality | ⚠️ Variable | Depends on Kling lip movements |
| Reliability | ✅ Good | Mature services |
| Latency | ⚠️ Moderate | +30-60s processing |

**Overall Feasibility: HIGH**

---

## Recommendations

### Primary Recommendation
Implement 3-stage audio pipeline:
1. Generate dialog script (predefined or GPT-generated)
2. Convert script to audio via ElevenLabs TTS
3. Sync audio to video via LatentSync

### Implementation Priority
1. Start with predefined script (simplest)
2. Test lip sync quality with Kling output
3. Iterate on script content based on results
4. Optionally add GPT-4 script generation later

### Risk Mitigation
- Keep initial scripts simple (exclamations, laughter)
- Test with multiple Picasso reference images
- Have fallback to silent video if sync fails

---

## References

### fal.ai Documentation
- LatentSync: https://fal.ai/models/fal-ai/latentsync
- Sync Lipsync v2.0: https://fal.ai/models/fal-ai/sync-lipsync/v2
- ElevenLabs TTS: https://fal.ai/models/fal-ai/elevenlabs/tts/multilingual-v2/api

### External Resources
- Sync Labs: https://sync.so/
- Sync Labs Docs: https://docs.sync.so/introduction
- ElevenLabs: https://elevenlabs.io/

### n8n Templates
- Talking Avatar: https://n8n.io/workflows/8378
- UGC Video Ads: https://n8n.io/workflows/10070
- Brand Marketing Shorts: https://n8n.io/workflows/5596

---

## Appendix: Alternative Approaches Considered

### A. Kling 2.6 with Image Composition
- Pre-compose user + Picasso images
- Use Kling 2.6 for video with native audio
- **Rejected:** Risk of identity loss in composition step

### B. Different Video Model (Veo 3, Sora 2)
- Switch to model with both identity and audio
- **Rejected:** Limited API access, migration effort

### C. External Audio Post-Processing
- Export video, process in dedicated audio tool
- **Rejected:** Breaks automation, manual intervention

---

*Research completed: 2026-01-10*
