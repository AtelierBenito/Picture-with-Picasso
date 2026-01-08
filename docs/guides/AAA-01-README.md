# Picture with Picasso

**Version:** 0.1.0 (Design Phase)
**Status:** Initial Design Complete
**Last Updated:** 2025-11-26

---

## Project Overview

An n8n automation workflow that lets users take a photo with friends at an art fair and receive a composited image/video placing them into a scene with Pablo Picasso.

### How It Works

1. **User sends photo** via Telegram, WhatsApp, or webhook
2. **Workflow randomly selects** a historical Picasso scene (beach, studio, café, gallery, etc.)
3. **Face swap composites** the user into the Picasso scene
4. **Optionally animates** the result with Picasso's voice
5. **Returns the result** to the user

---

## Technology Stack

| Component | Service | Purpose |
|-----------|---------|---------|
| Image Compositing | Replicate (easel/advanced-face-swap) | Place user in Picasso scene |
| Voice Generation | ElevenLabs | Spanish-accented Picasso voice |
| Video Animation | Replicate (bytedance/omni-human) | Animate composited image |
| Script Generation | OpenAI GPT-4o | Create dialogue |
| Input | Telegram / WhatsApp / Webhook | Receive user photos |
| Storage | Google Drive | Archive results |

---

## Project Status

### Completed
- [x] Platform research (AI restrictions on famous figures)
- [x] Technology stack selection
- [x] Workflow architecture design
- [x] Technical specification document

### In Progress
- [ ] n8n workflow JSON generation
- [ ] Picasso scene image collection

### Pending
- [ ] API account setup
- [ ] Voice configuration
- [ ] Testing and deployment

---

## Key Files

| File | Description |
|------|-------------|
| `PICTURE-WITH-PICASSO-WORKFLOW-DESIGN.md` | Complete technical specification |
| `AA-01-WHERE-WE-ARE-NOW.md` | Quick reference for current state |
| `AA-03-NEXT-STEPS.md` | Prioritized action items |

---

## Critical Technical Note

**Google Veo/Gemini/Imagen CANNOT be used** - These platforms block "prominent people" including historical figures. The solution uses **face-swap with real historical Picasso photos** instead of AI generation.

---

## Estimated Costs

| Mode | Cost per Use |
|------|--------------|
| Image only | ~$0.05 |
| With animation | ~$0.15-0.35 |

---

## Quick Start (Future)

```
1. Import workflow JSON to n8n
2. Configure credentials (Replicate, ElevenLabs, OpenAI)
3. Upload Picasso scene images to storage
4. Activate workflow
5. Send photo via Telegram/WhatsApp
```

---

*Project created: 2025-11-26*
