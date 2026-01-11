# ElevenLabs Integration Guide - Picture with Picasso

**Version:** v1.0-2026-01-10
**Document Type:** Integration Design & Setup Guide
**Purpose:** Complete ElevenLabs setup for voice, dialog, and ambient sound generation
**Related:** Audio Pipeline v3.1, Project Roadmap

---

## Executive Summary

ElevenLabs provides the complete audio solution for Picture with Picasso:
- **Picasso Voice**: Spanish-accented mature male TTS
- **Visitor Voices**: Dynamic voice selection (future)
- **Ambient Sounds**: 1920s artist studio atmosphere
- **MCP Server**: Direct Claude Code integration for development

**Key Finding**: An official ElevenLabs MCP server exists with 24 tools, enabling voice experimentation directly in Claude Code sessions.

---

## Table of Contents

1. [Account Setup](#1-account-setup)
2. [MCP Server Installation](#2-mcp-server-installation)
3. [Voice Design - Picasso](#3-voice-design---picasso)
4. [Sound Effects - Studio Ambience](#4-sound-effects---studio-ambience)
5. [Multi-Voice Dialog](#5-multi-voice-dialog)
6. [fal.ai API Integration](#6-falai-api-integration)
7. [Cost Analysis](#7-cost-analysis)
8. [n8n Workflow Integration](#8-n8n-workflow-integration)

---

## 1. Account Setup

### Step 1: Create ElevenLabs Account

1. Go to **https://elevenlabs.io**
2. Click **Sign Up** (top right)
3. Sign up with Google or email
4. Verify email address

### Step 2: Choose Plan

| Plan | Credits/Month | Cost | Features |
|------|---------------|------|----------|
| **Free** | 10,000 chars | $0 | Basic TTS, 3 custom voices |
| **Starter** | 30,000 chars | $5/mo | Voice cloning, 10 custom voices |
| **Creator** | 100,000 chars | $22/mo | Instant cloning, 30 custom voices |
| **Pro** | 500,000 chars | $99/mo | Professional cloning, 160 custom voices |

**Recommendation**: Start with **Free** tier for testing, upgrade to **Starter** ($5/mo) for production.

### Step 3: Get API Key

1. Log in to ElevenLabs dashboard
2. Click **Profile icon** (top right) → **Profile + API Key**
3. Click **Generate API Key** (or copy existing)
4. **Save securely** - you'll need this for:
   - MCP server configuration
   - n8n workflow configuration

### Step 4: Note Your Account Details

| Item | Where to Find | Save As |
|------|---------------|---------|
| API Key | Profile → API Key | `ELEVENLABS_API_KEY` |
| Account Email | Profile | Reference |
| Plan Type | Subscription | Reference |
| Credits Balance | Dashboard | Monitor |

---

## 2. MCP Server Installation

### Why Install the MCP Server?

The official ElevenLabs MCP server (24 tools) lets you:
- Test voice prompts directly in Claude Code
- Generate audio samples during development
- Experiment with voice settings before committing to workflow
- Create and preview voices without leaving your IDE

### Prerequisites

1. **Install uv** (Python package manager):
   ```bash
   # Windows (PowerShell)
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

   # macOS/Linux
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

2. **Verify installation**:
   ```bash
   uv --version
   ```

### Configuration for Claude Code

Add to your `C:\Users\BENIT\.claude.json`:

```json
{
  "mcpServers": {
    "ElevenLabs": {
      "command": "uvx",
      "args": ["elevenlabs-mcp"],
      "env": {
        "ELEVENLABS_API_KEY": "your-api-key-here",
        "ELEVENLABS_MCP_OUTPUT_MODE": "files",
        "ELEVENLABS_MCP_BASE_PATH": "C:\\Users\\BENIT\\Desktop"
      }
    }
  }
}
```

### Windows-Specific Notes

1. **Enable Developer Mode** in Claude Desktop:
   - Click hamburger menu (top left)
   - Select "Help" → "Enable Developer Mode"

2. **Path for uvx** - If you get "spawn uvx ENOENT" error:
   ```bash
   where uvx
   ```
   Use the full path (e.g., `C:\Users\BENIT\.local\bin\uvx.exe`) in configuration.

### Available MCP Tools (24 total)

| Category | Tools | Description |
|----------|-------|-------------|
| **TTS** | `text_to_speech`, `text_to_speech_stream` | Generate speech from text |
| **Voice Design** | `text_to_voice_design`, `create_voice_from_preview` | Create custom voices |
| **Voice Clone** | `voice_clone` | Clone voices from audio |
| **Sound Effects** | `sound_effect` | Generate ambient/SFX audio |
| **Audio Processing** | `audio_isolation` | Remove background noise |
| **Transcription** | `speech_to_text` | Convert audio to text |
| **Voice Management** | `list_voices`, `get_voice`, `search_voices` | Browse voice library |
| **Playback** | `play_audio` | Play generated audio locally |

### Test Your Installation

After restarting Claude Code, try:
```
"Use ElevenLabs to list available voices"
```

---

## 3. Voice Design - Picasso

### Creating Picasso's Voice

Picasso needs a distinctive voice:
- **Age**: 70s (mature, wise)
- **Gender**: Male
- **Accent**: Spanish (Mediterranean)
- **Tone**: Warm, energetic, welcoming, artistic
- **Emotion**: Joyful reunion energy

### Voice Design Prompt

```
A warm, expressive Spanish man in his 70s with a strong Mediterranean accent.
His voice carries the energy and passion of a lifelong artist - welcoming,
theatrical, and full of life. He speaks with the enthusiasm of reuniting with
an old friend, punctuated by hearty laughter. Think Pablo Picasso greeting
visitors to his studio in the south of France.
```

### Voice Design Parameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `prompt` | (above) | Voice characteristics |
| `loudness` | 0.6 | Slightly louder than default |
| `guidance_scale` | 4 | Balance creativity vs prompt adherence |
| `seed` | 42 | Reproducibility (use any number) |

### Test Script for Picasso

```
Ah, mi amigo! Welcome, welcome! Ha ha ha! It is so good to see you after all
these years! Come, come into my studio! Let me show you what I have been
creating. You look wonderful - life has been kind to you, yes?
```

### Using MCP to Create Voice

With ElevenLabs MCP installed, ask Claude Code:
```
Use ElevenLabs voice design to create a voice with this prompt:
"A warm, expressive Spanish man in his 70s with a strong Mediterranean accent..."
Then generate a preview with this text: "Ah, mi amigo! Welcome, welcome!"
```

### Saving the Voice

Once you find a voice you like:
1. Note the `generated_voice_id` from the preview
2. Create permanent voice: `create_voice_from_preview`
3. Save the `voice_id` to your workflow configuration

---

## 4. Sound Effects - Studio Ambience

### ElevenLabs Sound Effects Capabilities

ElevenLabs can generate:
- Ambient soundscapes (up to 30 seconds)
- Looping audio for continuous playback
- Environmental sounds
- Foley effects

### Studio Ambience Prompt

```
1920s Parisian artist studio ambience. Soft footsteps on worn wooden
floorboards, distant sounds of a bustling French street through open windows,
gentle breeze rustling papers and canvas, occasional sound of paintbrushes
against canvas, warm afternoon atmosphere with birds chirping faintly in the
distance. Vintage, cozy, creative workspace feeling.
```

### Sound Effect Parameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `text` | (prompt above) | Sound description |
| `duration_seconds` | 30 | Maximum length |
| `prompt_influence` | 0.5 | Balance creativity |
| `loop` | true | Seamless looping |

### Layering Strategy

For the final audio mix:

```
Layer 1: Picasso dialog (foreground)
   ↓
Layer 2: Studio ambience (background, -20dB)
   ↓
Final Mix → LatentSync for lip sync
```

### Alternative: Pre-made Ambient Sounds

If generated ambience isn't satisfactory:
- ElevenLabs Sound Library has free ambient sounds
- Categories: Environments, Ambience
- Search for: "art studio", "vintage room", "Paris cafe"

---

## 5. Multi-Voice Dialog

### Current Scope (Phase 2)

For Phase 2, Picasso speaks alone (monologue greeting):
- Single voice: Picasso
- No visitor dialog needed
- Simplifies lip sync

### Future Enhancement: Visitor Voices

For visitors to speak, we need:

1. **Gender/Age Detection**
   - Use GPT-4V or Claude Vision to analyze visitor photo
   - Detect: male/female, adult/child, approximate age
   - Return: voice selection recommendation

2. **Voice Library**
   Create a set of visitor voices:

   | Visitor Type | Voice Characteristics |
   |--------------|----------------------|
   | Adult Male | Warm, friendly male, neutral accent |
   | Adult Female | Warm, friendly female, neutral accent |
   | Child | Young, energetic, gender-neutral |
   | Elderly Male | Mature male, gentle |
   | Elderly Female | Mature female, gentle |

3. **Dialog Structure**
   ```json
   {
     "inputs": [
       {"text": "Picasso! It's so wonderful to see you!", "voice": "visitor_voice_id"},
       {"text": "Ah, my dear friend! Come in, come in!", "voice": "picasso_voice_id"},
       {"text": "Your studio is amazing!", "voice": "visitor_voice_id"},
       {"text": "Ha ha! Wait until you see my latest work!", "voice": "picasso_voice_id"}
     ]
   }
   ```

### TextToDialogueRequest API

ElevenLabs supports multi-speaker dialog in a single request:

```json
{
  "inputs": [
    {"text": "Dialog line 1", "voice": "voice_id_1"},
    {"text": "Dialog line 2", "voice": "voice_id_2"}
  ],
  "stability": 0.5,
  "seed": 42
}
```

---

## 6. fal.ai API Integration

### Available Endpoints

| Endpoint | Model | Use Case | Cost |
|----------|-------|----------|------|
| `fal-ai/elevenlabs/tts/eleven-v3` | Eleven v3 | Most expressive | Standard |
| `fal-ai/elevenlabs/tts/multilingual-v2` | Multilingual v2 | Stable, 29 languages | Standard |
| `fal-ai/elevenlabs/tts/turbo-v2.5` | Turbo v2.5 | Fast, low latency | $0.05/1k chars |

### API Request Format

```javascript
// Submit to queue
POST https://queue.fal.run/fal-ai/elevenlabs/tts/multilingual-v2
Headers:
  Authorization: Key YOUR_FAL_API_KEY
  Content-Type: application/json

Body:
{
  "text": "Ah, mi amigo! Welcome, welcome!",
  "voice": "your_picasso_voice_id",
  "stability": 0.5,
  "similarity_boost": 0.75,
  "style": 0.3,
  "speed": 1.0,
  "language_code": "es",
  "timestamps": true
}
```

### Response Format

```json
{
  "audio": {
    "url": "https://fal.media/files/xxx/output.mp3"
  },
  "timestamps": [
    {"text": "Ah", "start": 0.0, "end": 0.2},
    {"text": "mi", "start": 0.25, "end": 0.4},
    {"text": "amigo", "start": 0.45, "end": 0.9}
  ]
}
```

### Key Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | string | required | Text to speak |
| `voice` | string | "Rachel" | Voice name or ID |
| `stability` | float | 0.5 | Voice consistency (0-1) |
| `similarity_boost` | float | 0.75 | Voice matching (0-1) |
| `style` | float | 0 | Emotional expressiveness (0-1) |
| `speed` | float | 1.0 | Speech speed (0.7-1.2) |
| `language_code` | string | auto | ISO 639-1 code (e.g., "es") |
| `timestamps` | boolean | false | Return word-level timing |

### Timestamps for Lip Sync

The `timestamps` parameter is crucial for lip sync quality:
- Returns start/end times for each word
- Can be passed to LatentSync for precise alignment
- Helps match mouth movements to speech

---

## 7. Cost Analysis

### ElevenLabs Pricing

| Item | Cost | Notes |
|------|------|-------|
| Standard TTS | 1 credit/char | Multilingual v2 |
| Turbo/Flash | 0.5 credit/char | Faster models |
| Sound Effects | 40 credits/second | When duration specified |
| Voice Design | ~2000 credits | Per voice preview |

### Per-Video Cost Estimate

| Component | Characters/Duration | Credits | USD Equivalent |
|-----------|---------------------|---------|----------------|
| Picasso Dialog | ~100 chars | 100 | ~$0.03 |
| Ambient Sound | 10 seconds | 400 | ~$0.12 |
| **Per Video Total** | | ~500 | **~$0.15** |

### Monthly Projections

| Videos/Month | Credits Needed | Plan Recommendation |
|--------------|----------------|---------------------|
| 1-20 | 10,000 | Free |
| 21-60 | 30,000 | Starter ($5) |
| 61-200 | 100,000 | Creator ($22) |

---

## 8. n8n Workflow Integration

### Updated Audio Pipeline Architecture

```
                    ┌─────────────────────────┐
                    │   Kling O1 Silent Video │
                    └───────────┬─────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Picasso Voice │    │ Studio Ambience │    │ (Future)        │
│ ElevenLabs    │    │ Sound Effects   │    │ Visitor Voice   │
│ TTS v2        │    │ API             │    │ Detection       │
└───────┬───────┘    └────────┬────────┘    └─────────────────┘
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │    Audio Mixer      │
        │   (Cloudinary or    │
        │    fal.ai)          │
        └──────────┬──────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │    LatentSync       │
        │    Lip Sync         │
        └──────────┬──────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │   Final Video       │
        │   With Audio        │
        └─────────────────────┘
```

### Node Updates for Audio Pipeline v3.1

Add to `Workflow Configuration` node:

```json
{
  "falApiKey": "existing",
  "picassoVoiceId": "your_designed_voice_id",
  "dialogScript": "Ah, mi amigo! Welcome, welcome! Ha ha ha! It is so good to see you! Come, come!",
  "studioAmbienceUrl": "https://fal.media/files/xxx/studio_ambience.mp3",
  "elevenLabsModel": "multilingual-v2"
}
```

### n8n HTTP Request - ElevenLabs TTS

```javascript
// Node: Submit to ElevenLabs TTS
Method: POST
URL: https://queue.fal.run/fal-ai/elevenlabs/tts/multilingual-v2

Headers:
  Authorization: Key {{ $('Workflow Configuration').item.json.falApiKey }}
  Content-Type: application/json

Body:
{{ JSON.stringify({
  "text": $('Prepare Dialog Script').item.json.dialogScript,
  "voice": $('Workflow Configuration').item.json.picassoVoiceId,
  "stability": 0.5,
  "similarity_boost": 0.75,
  "style": 0.3,
  "speed": 1.0,
  "timestamps": true
}) }}
```

---

## 9. Implementation Checklist

### Phase 2 Preparation

- [ ] Create ElevenLabs account
- [ ] Get API key
- [ ] Install ElevenLabs MCP server
- [ ] Test MCP connection in Claude Code
- [ ] Design Picasso voice using Voice Design
- [ ] Save Picasso voice_id
- [ ] Generate studio ambience sound effect
- [ ] Save ambience audio URL
- [ ] Update Workflow Configuration node
- [ ] Test TTS endpoint via fal.ai

### Voice Design Session

When you're ready, we'll:
1. Use ElevenLabs MCP to iterate on Picasso voice
2. Generate multiple previews with different prompts
3. Select the best voice
4. Save to your ElevenLabs account
5. Record the voice_id for workflow use

---

## Related Documents

| Document | Purpose |
|----------|---------|
| Audio Pipeline v3.1 | Technical node specifications |
| Project Roadmap | Phase overview and timeline |
| Data Dictionary | Data elements for tracking |

---

## Sources

| Source | URL | Quality |
|--------|-----|---------|
| ElevenLabs MCP GitHub | github.com/elevenlabs/elevenlabs-mcp | High (Official) |
| ElevenLabs TTS Docs | elevenlabs.io/docs/overview/capabilities/text-to-speech | High (Official) |
| ElevenLabs Sound Effects | elevenlabs.io/docs/overview/capabilities/sound-effects | High (Official) |
| ElevenLabs Voices | elevenlabs.io/docs/overview/capabilities/voices | High (Official) |
| fal.ai ElevenLabs API | fal.ai/models/fal-ai/elevenlabs/tts/eleven-v3/api | High (Official) |

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-10 | 1.0 | Initial document from research |

---

*Document created: 2026-01-10*
*Research completed with: Firecrawl, Sequential Thinking MCP tools*
