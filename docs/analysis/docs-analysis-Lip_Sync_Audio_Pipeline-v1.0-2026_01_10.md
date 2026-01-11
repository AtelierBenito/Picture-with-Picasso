# Lip Sync & Audio Pipeline Architecture Analysis

**Version:** v1.0-2026-01-10
**Document Type:** Technical Analysis & Recommendations
**Purpose:** Answer critical architecture questions for audio/video synchronization

---

## Executive Summary

**Key Finding: SCRIPT-FIRST Architecture Required**

The workflow MUST be script-driven (audio-first), not video-first. LatentSync is audio-driven - it takes audio and applies lip movements to video. It does NOT analyze video mouth movements to generate matching audio.

**Recommended Order:**
1. Dialog Script (human/AI written)
2. Audio Generation (ElevenLabs)
3. Video Generation (Kling O1)
4. Lip Sync (LatentSync)

---

## Table of Contents

1. [Architecture Decision: Script-First vs Video-First](#1-architecture-decision)
2. [ElevenLabs Direct API vs fal.ai](#2-elevenlabs-direct-api)
3. [Multi-Person Voice Assignment](#3-multi-person-voice-assignment)
4. [Lip Sync Workflow: How LatentSync Works](#4-lip-sync-workflow)
5. [Should Kling Receive the Dialog Script?](#5-kling-dialog-prompts)
6. [Complete Recommended Workflow](#6-complete-workflow)
7. [Implementation Guidance](#7-implementation-guidance)

---

## 1. Architecture Decision: Script-First vs Video-First {#1-architecture-decision}

### The User's Question

> "Do we need to have Kling provide language text so then it can create the sounds because those figures are actually moving? That would be a better approach than having 11 labs try to decipher what words to fit into moving mouths."

### The Answer: Neither Tool Works That Way

**Critical Insight:** Neither LatentSync nor ElevenLabs can analyze video and generate matching dialog audio. The workflow MUST go in the opposite direction:

| Approach | How It Works | Feasibility |
|----------|--------------|-------------|
| Video-First | Generate video → analyze mouth → generate matching audio | **NOT POSSIBLE** with current tools |
| Script-First | Write script → generate audio → generate video → apply lip sync | **CORRECT APPROACH** |

### Why Script-First is the Only Option

1. **ElevenLabs** converts TEXT → AUDIO (not video → audio)
2. **LatentSync** applies AUDIO → VIDEO lip movements (not video → audio)
3. **Kling O1** generates video from prompts (can include dialog timing hints)

The dialog script is the **source of truth** for the entire pipeline.

---

## 2. ElevenLabs Direct API vs fal.ai {#2-elevenlabs-direct-api}

### User Preference

> "I would prefer to access ElevenLabs directly. Instead of through fal.ai because I expect to be using it in the future."

### Recommendation: Use Direct API

| Factor | ElevenLabs Direct | fal.ai Wrapper |
|--------|-------------------|----------------|
| Latency | Lower (no intermediary) | Higher (queue-based) |
| Features | Full access | May be limited |
| Billing | Unified ElevenLabs account | Separate fal.ai billing |
| Future-proof | Immediate new features | Depends on fal.ai updates |
| Complexity | Simpler (synchronous) | More complex (queue/polling) |

### Direct API Configuration for n8n

```javascript
// HTTP Request Node
Method: POST
URL: https://api.elevenlabs.io/v1/text-to-speech/{{ $json.voiceId }}

Headers:
  xi-api-key: {{ $credentials.elevenLabsApiKey }}
  Content-Type: application/json

Body:
{
  "text": "{{ $json.dialogText }}",
  "model_id": "eleven_multilingual_v2",
  "voice_settings": {
    "stability": 0.5,
    "similarity_boost": 0.75,
    "style": 0.3
  }
}

Response: Binary audio file (MP3/WAV)
```

### API Endpoint Summary

| Endpoint | Purpose |
|----------|---------|
| `POST /v1/text-to-speech/{voice_id}` | Generate speech from text |
| `POST /v1/sound-generation` | Generate sound effects |
| `POST /v1/voice-generation/generate-voice` | Create custom voice from prompt |
| `GET /v1/voices` | List available voices |

---

## 3. Multi-Person Voice Assignment {#3-multi-person-voice-assignment}

### User's Question

> "It's going to need to count heads, their sex, and does it create random sounds and random voices?"

### How It Works

ElevenLabs does NOT automatically:
- Count people in images
- Detect gender/age
- Generate random voices

**We must build this logic ourselves:**

### Voice Assignment Pipeline

```
Visitor Photo
     │
     ▼
┌─────────────────────────┐
│   Face Detection API    │
│   (Cloudinary, Azure,   │
│    Google Vision)       │
└───────────┬─────────────┘
            │
            ▼
     ┌──────────────┐
     │  For Each    │
     │  Detected    │
     │    Face      │
     └──────┬───────┘
            │
            ▼
┌─────────────────────────┐
│   Analyze Attributes    │
│   - Gender (M/F)        │
│   - Age range           │
│   - Position in image   │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   Select Voice from     │
│   Pre-Created Library   │
└───────────┬─────────────┘
            │
            ▼
     Voice Assignment
     Complete
```

### Voice Library Strategy

Pre-create 8-10 voices in ElevenLabs:

| Voice ID | Type | Use Case |
|----------|------|----------|
| `picasso_voice` | Male, 70s, Spanish | Fixed - always Picasso |
| `visitor_male_young` | Male, 20-35, neutral | Young male visitors |
| `visitor_male_mature` | Male, 40-60, warm | Mature male visitors |
| `visitor_female_young` | Female, 20-35, friendly | Young female visitors |
| `visitor_female_mature` | Female, 40-60, warm | Mature female visitors |
| `visitor_child` | Child, gender-neutral | Young visitors |

### Face Detection Options

| Service | MCP Available? | Capabilities |
|---------|----------------|--------------|
| **Cloudinary Analysis** | Yes (`mcp__cloudinary-analysis__*`) | Human anatomy detection |
| Azure Face API | No (HTTP Request) | Face count, gender, age, emotion |
| Google Cloud Vision | No (HTTP Request) | Face detection, likelihood scores |
| Amazon Rekognition | No (HTTP Request) | Face detection, attributes |

**Recommendation:** Use Cloudinary since MCP is already configured:
- `mcp__cloudinary-analysis__analyze-human-anatomy`

---

## 4. Lip Sync Workflow: How LatentSync Works {#4-lip-sync-workflow}

### Critical Understanding

**LatentSync is AUDIO-DRIVEN:**

```
INPUT:
┌─────────────┐     ┌─────────────┐
│    Video    │  +  │    Audio    │
│  (any mouth │     │  (speech)   │
│  movements) │     │             │
└──────┬──────┘     └──────┬──────┘
       │                   │
       └─────────┬─────────┘
                 │
                 ▼
       ┌─────────────────┐
       │   LatentSync    │
       │                 │
       │  1. Whisper     │ ← Extracts audio embeddings
       │  2. Face Detect │ ← Finds mouth region
       │  3. Stable Diff │ ← Regenerates mouth area
       └────────┬────────┘
                │
                ▼
OUTPUT:
┌─────────────────────────┐
│   Video with NEW lip    │
│   movements matching    │
│   the provided audio    │
└─────────────────────────┘
```

### Key Insights

1. **Input video mouth movements are IRRELEVANT** - LatentSync replaces them
2. **Audio file DRIVES the output** - lips match the audio, not vice versa
3. **Works on static faces** - even images with closed mouths work
4. **Quality depends on face visibility** - clear, forward-facing faces work best

### What This Means for Our Pipeline

The Kling O1 video quality matters for:
- Overall scene composition
- Character expressions and body language
- Background and lighting

The Kling O1 video does NOT need to have:
- Accurate lip sync (will be replaced)
- Correct mouth movements (will be replaced)
- Silent characters (movement is fine)

---

## 5. Should Kling Receive the Dialog Script? {#5-kling-dialog-prompts}

### Short Answer: Include Timing Hints, Not Exact Words

### Kling 2.6 Pro Capabilities

Kling CAN include dialog in prompts:
```
"The character says: 'Welcome to my studio!' with a warm smile"
```

This creates approximate mouth movements - not precise lip sync.

### Recommended Approach

**DO include:**
- Dialog TIMING (when character speaks)
- Emotional cues (warmly, excitedly, thoughtfully)
- Physical actions during speech (gesturing, walking)

**DON'T worry about:**
- Exact word-to-mouth matching
- Precise syllable timing
- Perfect lip formations

### Example Kling Prompt Strategy

**Instead of:**
```
The artist says: "Ah, mi amigo! Welcome, welcome! It is so good to see you!"
```

**Use:**
```
Picasso greets the visitor warmly for 5 seconds, his face animated with joy.
He gestures expressively with his hands while speaking, then pauses to
look at the visitor with a knowing smile. His mouth moves naturally as
he speaks, eyes crinkling with genuine happiness.
```

This gives Kling:
- Natural motion cues
- Timing guidance
- Emotional direction

Without requiring:
- Perfect word-to-lip matching (LatentSync handles this)

---

## 6. Complete Recommended Workflow {#6-complete-workflow}

### Phase 1: Pre-Production (Script & Audio)

```
┌─────────────────────────────────────────────────────────────┐
│                    PHASE 1: AUDIO FIRST                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: User uploads visitor photo                          │
│       │                                                      │
│       ▼                                                      │
│  Step 2: Face Detection (Cloudinary)                         │
│       │   → Count faces                                      │
│       │   → Detect gender/age per face                       │
│       │                                                      │
│       ▼                                                      │
│  Step 3: Generate Dialog Script (GPT-4/Claude)               │
│       │   → Personalized greeting                            │
│       │   → Speaker tags: [PICASSO], [VISITOR_1], etc.       │
│       │                                                      │
│       ▼                                                      │
│  Step 4: Voice Assignment                                    │
│       │   → Picasso = fixed voice                            │
│       │   → Visitors = selected from library by attributes   │
│       │                                                      │
│       ▼                                                      │
│  Step 5: Generate Audio (ElevenLabs Direct API)              │
│       │   → Call TTS for each speaker segment                │
│       │   → Concatenate audio tracks                         │
│       │                                                      │
│       ▼                                                      │
│  Step 6: Generate Ambient Sound (ElevenLabs Sound Effects)   │
│       │   → Studio ambience                                  │
│       │                                                      │
│       ▼                                                      │
│  Step 7: Mix Audio                                           │
│       │   → Dialog (foreground)                              │
│       │   → Ambience (background, -20dB)                     │
│       │                                                      │
│       ▼                                                      │
│  OUTPUT: Complete audio track                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Phase 2: Video Generation

```
┌─────────────────────────────────────────────────────────────┐
│                   PHASE 2: VIDEO SECOND                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 8: Generate Kling Prompt                               │
│       │   → Include timing hints from script                 │
│       │   → Describe motion, emotion, gestures               │
│       │   → Reference duration from audio                    │
│       │                                                      │
│       ▼                                                      │
│  Step 9: Submit to Kling O1                                  │
│       │   → Image + prompt                                   │
│       │   → Wait for completion                              │
│       │                                                      │
│       ▼                                                      │
│  OUTPUT: Silent video with natural motion                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Phase 3: Lip Sync & Assembly

```
┌─────────────────────────────────────────────────────────────┐
│                   PHASE 3: LIP SYNC THIRD                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 10: Submit to LatentSync                               │
│       │   → Video file (from Kling)                          │
│       │   → Audio file (dialog only, not ambience)           │
│       │                                                      │
│       ▼                                                      │
│  Step 11: LatentSync processes                               │
│       │   → Extracts audio features                          │
│       │   → Detects faces                                    │
│       │   → Regenerates lip movements                        │
│       │                                                      │
│       ▼                                                      │
│  Step 12: Final Assembly                                     │
│       │   → Lip-synced video                                 │
│       │   → Add ambient audio track                          │
│       │                                                      │
│       ▼                                                      │
│  OUTPUT: Final video with perfect lip sync                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Implementation Guidance {#7-implementation-guidance}

### ElevenLabs Direct API Setup

**1. Account Setup:**
- Go to elevenlabs.io
- Create account
- Get API key from Profile

**2. n8n Credentials:**
- Create "Header Auth" credential
- Name: `xi-api-key`
- Value: Your API key

**3. HTTP Request Node:**
```json
{
  "method": "POST",
  "url": "https://api.elevenlabs.io/v1/text-to-speech/{{ voice_id }}",
  "headers": {
    "xi-api-key": "{{ api_key }}",
    "Content-Type": "application/json"
  },
  "body": {
    "text": "Dialog text here",
    "model_id": "eleven_multilingual_v2",
    "voice_settings": {
      "stability": 0.5,
      "similarity_boost": 0.75
    }
  }
}
```

### Face Detection with Cloudinary

```javascript
// Use Cloudinary Analysis MCP or HTTP Request
POST https://api.cloudinary.com/v1_1/{cloud_name}/image/analyze
Body: {
  "public_id": "visitor_photo",
  "analysis": ["human_anatomy"]
}
```

### Voice Selection Logic (n8n Function Node)

```javascript
// Based on face detection results
function selectVoice(faceData) {
  const voices = {
    male_young: "voice_id_1",
    male_mature: "voice_id_2",
    female_young: "voice_id_3",
    female_mature: "voice_id_4",
    child: "voice_id_5"
  };

  const gender = faceData.gender; // 'male' or 'female'
  const age = faceData.age;

  if (age < 13) return voices.child;
  if (gender === 'male') {
    return age < 40 ? voices.male_young : voices.male_mature;
  }
  return age < 40 ? voices.female_young : voices.female_mature;
}
```

---

## Summary of Answers

| Question | Answer |
|----------|--------|
| ElevenLabs direct or fal.ai? | **Direct API** - simpler, full features, future-proof |
| How handle multiple people? | **Face detection API** → voice library selection |
| Does 11Labs create random voices? | **No** - pre-create voice library, select based on attributes |
| Lip sync from video movements? | **Not possible** - LatentSync is audio-driven |
| Should Kling get dialog script? | **Timing hints yes**, exact words not necessary |
| Best workflow order? | **Script → Audio → Video → Lip Sync** |

---

## Sources

| Source | URL | Quality |
|--------|-----|---------|
| LatentSync GitHub | github.com/bytedance/LatentSync | High (Official) |
| ElevenLabs API Docs | elevenlabs.io/docs/api-reference | High (Official) |
| Kling Avatar v2 Guide | fal.ai/learn/devs/kling-avatar-v2-prompt-guide | High |
| Artlist Kling Tutorial | artlist.io/blog/talking-characters-kling-ai | Medium |

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-10 | 1.0 | Initial analysis document |

---

*Document created: 2026-01-10*
*Research tools: Firecrawl, Sequential Thinking*
