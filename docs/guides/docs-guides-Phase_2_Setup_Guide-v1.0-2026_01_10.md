# Phase 2 Setup Guide - Interactive Audio Pipeline

**Version:** v1.0-2026-01-10
**Purpose:** Pre-session preparation checklist for Phase 2 build
**Estimated Setup Time:** 45-60 minutes
**Build Session Time:** 7-10 hours (can be split across multiple sessions)

---

## Overview

Before we build the Interactive Audio Pipeline, you need to set up accounts and credentials for:

1. ElevenLabs (voice generation)
2. Azure Face API (face recognition)
3. OpenRouter (AI script generation)
4. OpenAI (content moderation)

This guide walks you through each step.

---

## Checklist Summary

| Task | Time | Priority | Status |
|------|------|----------|--------|
| [ ] ElevenLabs account + API key | 5 min | Required | |
| [ ] Create Picasso voice | 15 min | Required | |
| [ ] Create your voice clone | 15 min | Required | |
| [ ] Azure account + Face API | 10 min | Required | |
| [ ] Train BENIT face reference | 10 min | Required | |
| [ ] OpenRouter account + API key | 5 min | Required | |
| [ ] OpenAI account + API key | 5 min | Required | |
| [ ] Gather reference photos | 5 min | Required | |
| [ ] Install ElevenLabs MCP (optional) | 5 min | Optional | |

---

## Step 1: ElevenLabs Setup

### 1.1 Create Account

1. Go to **https://elevenlabs.io**
2. Click **Sign Up** (top right)
3. Sign up with Google or email
4. Verify your email

### 1.2 Get API Key

1. Log in to ElevenLabs
2. Click your **Profile icon** (top right)
3. Select **Profile + API Key**
4. Click **Generate API Key** (or copy existing)
5. **Save this key securely** - you'll need it for n8n

```
Your ElevenLabs API Key: ________________________________
```

### 1.3 Create Picasso Voice (Voice Design)

1. Go to **Voices** → **Voice Library** → **Add Voice**
2. Select **Voice Design**
3. Enter this prompt:

```
A warm, expressive Spanish man in his 70s with a strong Mediterranean accent.
His voice carries the energy and passion of a lifelong artist - welcoming,
theatrical, and full of life. He speaks with the enthusiasm of reuniting
with an old friend, punctuated by hearty laughter. Deep, resonant voice
with artistic flair.
```

4. Generate preview
5. If satisfied, click **Create Voice**
6. Name it: `Picasso_Voice`
7. **Save the Voice ID:**

```
Picasso Voice ID: ________________________________
```

### 1.4 Create Your Voice Clone

1. Go to **Voices** → **Voice Library** → **Add Voice**
2. Select **Instant Voice Clone**
3. **Record or upload audio of yourself:**
   - 1-3 minutes of natural speech
   - Clean audio, no background noise
   - Speak in your normal voice
   - Include some emotion/expression

4. Upload the audio file(s)
5. Click **Add Voice**
6. Name it: `BENIT_Voice`
7. **Save the Voice ID:**

```
BENIT (Your) Voice ID: ________________________________
```

### 1.5 Test Your Voices

Test both voices with sample text:
- Picasso: "Ah, mi amigo! Welcome to my studio!"
- Your voice: "Maestro! It's wonderful to see you!"

---

## Step 2: Azure Face API Setup

### 2.1 Create Azure Account

1. Go to **https://portal.azure.com**
2. Sign up for free account (requires credit card, but won't charge for free tier)
3. Complete verification

### 2.2 Create Face API Resource

1. In Azure Portal, click **Create a resource**
2. Search for **Face**
3. Click **Create**
4. Fill in:
   - **Subscription:** Your subscription
   - **Resource group:** Create new → `picasso-resources`
   - **Region:** `East US` (or closest to you)
   - **Name:** `picasso-face-api`
   - **Pricing tier:** `Free F0` (30,000 calls/month)
5. Click **Review + Create** → **Create**
6. Wait for deployment (~1 minute)

### 2.3 Get API Key and Endpoint

1. Go to your new Face API resource
2. Click **Keys and Endpoint** (left sidebar)
3. Copy **KEY 1** and **Endpoint**

```
Azure Face API Key: ________________________________
Azure Face Endpoint: ________________________________
(Example: https://eastus.api.cognitive.microsoft.com/)
```

### 2.4 Create PersonGroup for BENIT

We'll do this together in the build session, but prepare:

**Reference Photos of Yourself (3-5 photos):**
- Clear face, good lighting
- Different angles (front, slight left, slight right)
- No sunglasses or face obstructions
- Recent photos

Save these photos in a folder:
```
Picasso Images/BENIT_Reference/
    photo1.jpg
    photo2.jpg
    photo3.jpg
```

---

## Step 3: OpenRouter Setup

### 3.1 Create Account

1. Go to **https://openrouter.ai**
2. Click **Sign In** (top right)
3. Sign up with Google or email

### 3.2 Get API Key

1. Go to **https://openrouter.ai/keys**
2. Click **Create Key**
3. Name it: `PicassoProject`
4. Copy the key

```
OpenRouter API Key: ________________________________
```

### 3.3 Add Credits (Optional)

- OpenRouter offers some free credits
- For production, add $5-10 prepaid credits
- Grok 4.1 Fast is very cheap (~$0.0001 per script)

---

## Step 4: OpenAI Setup (for Moderation)

### 4.1 Create Account

1. Go to **https://platform.openai.com**
2. Sign up or log in
3. Go to **API Keys**

### 4.2 Get API Key

1. Click **Create new secret key**
2. Name it: `PicassoModeration`
3. Copy the key

```
OpenAI API Key: ________________________________
```

**Note:** The Moderation API is FREE to use, but you need an API key.

---

## Step 5: Prepare Reference Materials

### 5.1 Your Reference Photos (for Face Recognition)

Create folder and add 3-5 clear photos of yourself:
```
C:\Users\BENIT\OneDrive\AI Studios\Claude Code\Projects\Picture with Picasso\
    Picasso Images\
        BENIT_Reference\
            benit_1.jpg
            benit_2.jpg
            benit_3.jpg
```

### 5.2 Test Visitor Photos

Prepare 2-3 test photos for testing:
- One with you in it (to test BENIT recognition)
- One with someone else (to test stranger flow)
- One with multiple people (to test multi-face detection)

---

## Step 6: Optional - Install ElevenLabs MCP

For voice experimentation directly in Claude Code:

### 6.1 Install uv (if not already installed)

```powershell
# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 6.2 Add to Claude Configuration

Add to `C:\Users\BENIT\.claude.json`:

```json
{
  "mcpServers": {
    "ElevenLabs": {
      "command": "uvx",
      "args": ["elevenlabs-mcp"],
      "env": {
        "ELEVENLABS_API_KEY": "your-elevenlabs-api-key",
        "ELEVENLABS_MCP_OUTPUT_MODE": "files",
        "ELEVENLABS_MCP_BASE_PATH": "C:\\Users\\BENIT\\Desktop"
      }
    }
  }
}
```

### 6.3 Restart Claude Code

Close and reopen Claude Code to load the MCP server.

---

## Credential Summary Card

Fill this out and keep handy for the build session:

```
╔════════════════════════════════════════════════════════════════╗
║                    PHASE 2 CREDENTIALS                          ║
╠════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  ELEVENLABS                                                      ║
║  API Key: _______________________________________________       ║
║  Picasso Voice ID: ______________________________________       ║
║  BENIT Voice ID: ________________________________________       ║
║                                                                  ║
║  AZURE FACE API                                                  ║
║  Endpoint: ______________________________________________       ║
║  API Key: _______________________________________________       ║
║                                                                  ║
║  OPENROUTER                                                      ║
║  API Key: _______________________________________________       ║
║                                                                  ║
║  OPENAI (Moderation)                                             ║
║  API Key: _______________________________________________       ║
║                                                                  ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Build Session Agenda

Once setup is complete, our build session will cover:

### Session 1: Phase 2A - Core Audio (2-3 hours)
- [ ] Add Picasso TTS node (ElevenLabs)
- [ ] Add ambient sound generation
- [ ] Add audio mixing node
- [ ] Add LatentSync lip sync nodes
- [ ] Test end-to-end with static script

### Session 2: Phase 2B - Face Recognition (2 hours)
- [ ] Create PersonGroup in Azure
- [ ] Train with your reference photos
- [ ] Add face detection node
- [ ] Add face identification node
- [ ] Add voice assignment logic
- [ ] Test BENIT recognition

### Session 3: Phase 2C - Interactive Input (2-3 hours)
- [ ] Add content moderation (input)
- [ ] Add Grok script generation
- [ ] Add content moderation (output)
- [ ] Connect user message to script
- [ ] Test full interactive flow

### Session 4: Phase 2D - Polish (1-2 hours)
- [ ] Add visitor joy sounds
- [ ] Refine audio mixing
- [ ] Error handling
- [ ] Full testing
- [ ] Documentation update

---

## Troubleshooting

### ElevenLabs Voice Not Working
- Check API key is correct
- Verify voice_id exists in your account
- Check you have credits remaining

### Azure Face API Errors
- Verify endpoint URL includes region
- Check API key is from correct resource
- Ensure PersonGroup is created and trained

### OpenRouter Not Responding
- Check API key is valid
- Verify account has credits
- Try different model if Grok is unavailable

### Content Moderation False Positives
- Review the flagged categories
- Adjust input text if legitimate
- Contact OpenAI if systematic issues

---

## Ready Check

Before starting the build session, confirm:

- [ ] ElevenLabs account created and API key saved
- [ ] Picasso voice created and Voice ID saved
- [ ] Your voice cloned and Voice ID saved
- [ ] Azure account created and Face API resource deployed
- [ ] Azure Face API key and endpoint saved
- [ ] Reference photos of yourself prepared (3-5 photos)
- [ ] OpenRouter account created and API key saved
- [ ] OpenAI account created and API key saved
- [ ] Test visitor photos prepared

**When ready, message:** "Ready to build Phase 2!"

---

## Questions?

If you encounter issues during setup:
1. Note the specific error message
2. Which step you're on
3. What you tried

We'll troubleshoot together in the build session.

---

*Guide created: 2026-01-10*
*Estimated total setup time: 45-60 minutes*
