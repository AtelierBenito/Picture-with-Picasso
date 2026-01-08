# Session Summary - 2025-11-26

## Session Overview

**Date:** 2025-11-26
**Duration:** ~45 minutes
**Focus:** Initial project design, research, and technical specification

---

## Accomplishments

### 1. Platform Research & Validation
- Researched AI platform policies for generating content with famous/historical figures
- Discovered Google Veo/Gemini/Imagen **block** "prominent people" - cannot use
- Found Flux (Black Forest Labs) trained to avoid celebrities - limited use
- Validated Replicate face-swap APIs work (uses real photos, doesn't generate)
- Confirmed ElevenLabs supports historical figure voice generation

### 2. Technology Stack Selection
Selected and validated complete tech stack:
- **Image Compositing:** Replicate (easel/advanced-face-swap)
- **Voice Generation:** ElevenLabs (eleven_multilingual_v2)
- **Video Animation:** Replicate (bytedance/omni-human)
- **Script Generation:** OpenAI GPT-4o
- **Input Triggers:** Telegram/WhatsApp/Webhook
- **Storage:** Google Drive

### 3. Workflow Architecture Design
Designed complete n8n workflow with 5 sections:
1. Trigger & Input (image receipt)
2. Random Picasso Scene Selection (Code node)
3. Face Swap Compositing (Replicate API + polling)
4. Optional Animation Branch (voice + video)
5. Delivery (return to user)

### 4. Documentation Created
- `PICTURE-WITH-PICASSO-WORKFLOW-DESIGN.md` - Complete 400+ line technical specification

---

## Files Created This Session

| File | Purpose |
|------|---------|
| `PICTURE-WITH-PICASSO-WORKFLOW-DESIGN.md` | Complete workflow design, prompt, and technical specs |

---

## Files Modified This Session

None (new project)

---

## Key Decisions Made

### Decision 1: Use Face Swap Instead of AI Generation
**What:** Composite users into REAL historical Picasso photos rather than AI-generating Picasso
**Why:** Google/major AI platforms block famous figure generation
**Impact:** More authentic results using actual historical photos

### Decision 2: Replicate as Primary AI Platform
**What:** Use Replicate for both face swap (easel/advanced-face-swap) and video (bytedance/omni-human)
**Why:** Single platform, good API, supports the models we need
**Impact:** Simplified credential management, consistent API patterns

### Decision 3: ElevenLabs for Picasso Voice
**What:** Use ElevenLabs with Spanish-accented historical voice
**Why:** ElevenLabs explicitly supports historical/educational purposes, has Iconic Marketplace
**Impact:** Authentic-sounding Picasso voice for animations

---

## Technical Insights

1. **Async API Pattern:** Replicate uses create-prediction → wait → poll-status → get-result pattern. Need Wait nodes and IF loops in n8n.

2. **Existing Templates:** Found n8n community templates (#6876, #4056, #5100) that demonstrate similar image-to-video workflows - can reference for implementation.

3. **Cost Efficiency:** Image-only mode is very cheap (~$0.05), animation adds ~$0.10-0.30 for video generation.

---

## What's Next

**Immediate Priority:**
1. Generate actual n8n workflow JSON from the design document

**High Priority:**
2. Gather 5+ public domain Picasso photographs for scenes
3. Create API accounts (Replicate, ElevenLabs, OpenAI)
4. Test face swap quality with sample images

---

## Blockers

None currently. Project is in design phase, ready to move to implementation.

---

## Research Sources Used

- n8n MCP Server (node documentation, templates)
- Firecrawl web search (platform policies)
- Sequential thinking tool (structured analysis)
- Context7 documentation (validation)

---

*Session closed: 2025-11-26*
