# Picture with Picasso - Core Requirements Checklist

**Version:** 1.0
**Last Updated:** 2026-01-11
**Purpose:** Non-negotiable requirements for tool/model evaluation

---

## How to Use

Before recommending ANY tool change, validate against ALL requirements below. A model must meet every requirement marked **REQUIRED** to be considered.

---

## Core Requirements

### 1. Multi-Image Composition (REQUIRED)

| Requirement | Details |
|-------------|---------|
| Input | 2 images: visitor photo + Picasso environment |
| Output | Single composed scene with both elements |
| Method | Must accept 2+ image inputs natively OR via element/reference syntax |

**Validation Question:** Can the model accept two separate images and combine them into one output?

**Current Solution:** Kling O1 Elements (`@Element1`, `@Element2`)

---

### 2. Identity Preservation (REQUIRED)

| Requirement | Details |
|-------------|---------|
| Visitor face | Must preserve visitor's facial identity |
| Picasso likeness | Must maintain recognizable Picasso appearance |
| No face swap | Cannot generate new/different faces |

**Validation Question:** Does the model preserve the exact faces from input images, or does it generate new interpretations?

**Known Failures:** Reve Remix (generates new faces, does not preserve identity)

---

### 3. Celebrity/Historical Figure Handling (REQUIRED)

| Requirement | Details |
|-------------|---------|
| Subject | Pablo Picasso (deceased 1973) |
| Usage | Artistic/educational context |
| Risk tolerance | Medium - must not be blocked by safety filters |

**Validation Question:** Does the model allow generation of recognizable historical figures?

---

### 4. Prompt Capacity (REQUIRED)

| Requirement | Details |
|-------------|---------|
| Minimum | 2,500 characters (current prompt size after v1.5 fix) |
| Preferred | 3,000+ characters for future expansion |
| Includes | Scene description, character actions, style directives |

**Validation Question:** What is the prompt character limit? Is it sufficient for detailed scene control?

**Known Limits:**
- Kling O1: 2,500 chars (sufficient)
- WAN 2.6 via fal.ai: 800 chars (insufficient)

---

### 5. API Accessibility (REQUIRED)

| Requirement | Details |
|-------------|---------|
| Platform | Must be available via fal.ai OR direct API |
| Integration | Compatible with n8n HTTP Request node |
| Authentication | API key based |

**Validation Question:** Is there a documented API endpoint accessible from n8n?

---

## Desired Requirements

### 6. Native Audio Generation (DESIRED)

| Requirement | Details |
|-------------|---------|
| Benefit | Would eliminate Phase 2 audio pipeline |
| Current plan | Separate ElevenLabs + LatentSync pipeline |

**Validation Question:** Does the model generate audio natively, or require separate audio processing?

---

### 7. Lip Sync Capability (DESIRED)

| Requirement | Details |
|-------------|---------|
| Benefit | Character speech matches audio |
| Current plan | LatentSync post-processing |

**Validation Question:** Does the model support audio-driven lip sync?

---

## Model Evaluation Template

When evaluating a new model, copy and complete:

```
## [Model Name] Evaluation

Date: YYYY-MM-DD
Source: [Changelog/Announcement URL]

### Required Capabilities

| # | Requirement | Supported? | Evidence |
|---|-------------|------------|----------|
| 1 | Multi-image composition | | |
| 2 | Identity preservation | | |
| 3 | Celebrity handling | | |
| 4 | Prompt capacity (2,500+) | | |
| 5 | API accessibility | | |

### Desired Capabilities

| # | Requirement | Supported? | Evidence |
|---|-------------|------------|----------|
| 6 | Native audio | | |
| 7 | Lip sync | | |

### Recommendation

[ ] PROCEED - All required capabilities confirmed
[ ] REJECT - Missing required capability: ___
[ ] INVESTIGATE - Need more information on: ___
```

---

## Evaluated Models

| Model | Date | Result | Reason |
|-------|------|--------|--------|
| Kling O1 Elements | 2026-01-06 | **APPROVED** | Meets all required capabilities |
| Reve Remix | 2026-01-06 | REJECTED | Fails #2 (identity preservation) |
| WAN 2.6 | 2026-01-11 | REJECTED | Fails #1 (single image only), #4 (800 char limit) |
| Grok Imagine | 2026-01-11 | REJECTED | Fails #1 (single image only) |

---

## Grok Imagine Evaluation

**Date:** 2026-01-11
**Source:** [Kie.ai Grok Imagine](https://kie.ai/grok-imagine), [xAI Docs](https://docs.x.ai/docs/guides/image-generations)

### Required Capabilities

| # | Requirement | Supported? | Evidence |
|---|-------------|------------|----------|
| 1 | Multi-image composition | **NO** | "only one image is supported" for video generation |
| 2 | Identity preservation | Unknown | No documentation on face preservation |
| 3 | Celebrity handling | Likely YES | "Spicy mode" suggests flexible content policy |
| 4 | Prompt capacity (2,500+) | **YES** | 5,000 character limit |
| 5 | API accessibility | **YES** | Available via Kie.ai API |

### Desired Capabilities

| # | Requirement | Supported? | Evidence |
|---|-------------|------------|----------|
| 6 | Native audio | **YES** | "synchronized audio and ambient sound effects" |
| 7 | Lip sync | Unknown | No documentation found |

### Recommendation

[X] REJECT - Missing required capability: #1 Multi-image composition

### Notes

- Grok Imagine is xAI's (Elon Musk) Aurora engine
- Generates 6-15 second videos with audio
- Three modes: Normal, Fun, Spicy (Spicy disabled for external images)
- Strong on prompt capacity (5,000 chars) and audio
- **Critical limitation:** Image-to-Video accepts only ONE image input

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-11 | 1.0 | Initial checklist created after WAN 2.6 validation failure |

---

*Reference: LL-002 - Validate Model Capabilities Before Tool Change*
