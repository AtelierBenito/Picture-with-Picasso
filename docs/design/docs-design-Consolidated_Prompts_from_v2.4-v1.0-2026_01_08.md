# Consolidated Prompts from Workflow v2.4

**Version:** v1.0-2026-01-08
**Source Workflow:** `Picture with Picasso -Wan 2.5-v2.4.json`
**Purpose:** Extract and preserve existing prompts for v3.0 Kling O1 migration

---

## Overview

This document extracts and preserves the existing prompts from workflow v2.4 to inform the design of the new Kling O1 prompt. These prompts represent the accumulated creative intent developed through multiple iterations.

---

## Source Reference

| Prompt | Node | Workflow File |
|--------|------|---------------|
| Reve Remix Prompt | `Send to Reve Remix` | `Picture with Picasso -Wan 2.5-v2.4.json` |
| Video Prompt | `Prepare Video Prompt` | `Picture with Picasso -Wan 2.5-v2.4.json` |

---

## Why Consolidation is Required

1. **Architecture Change:** v3.0 eliminates the intermediate image composition step, so two separate prompts must become one unified prompt
2. **API Difference:** Kling O1 uses @Element references instead of image array indices
3. **Preservation:** The Reve Remix prompt contains valuable details about scaling, proportions, and positioning that should not be lost
4. **Intent Continuity:** Both prompts express the creative vision that must carry forward to v3.0

---

## Extracted Prompt 1: Reve Remix (Image Composition)

**Node:** `Send to Reve Remix`
**Purpose:** Compose user image with Picasso artwork while preserving identity and style

```
You receive two images. Image 1 is a photo of the user or users. Image 2 is a Picasso in his environment that must be used as the base canvas. Use Image 2 as the main scene and keep its background, composition, aspect ratio, colors, and Picasso style intact. From Image 1, detect and extract only the foreground person or people, removing their original background completely. Restyle the extracted person or people so they visually match the Picasso style of Image 2 while remaining recognizable. Place them into Image 2 so they appear to be very close friends with the existing Picasso character or characters: standing or sitting close together, facing toward each other with warm, playful, emotionally connected body language, as if they are joyfully chumming around and having fun. Scale the inserted person or people so they look like realistically sized adults standing next to the Picasso character or characters. Imagine the Picasso character is an adult about 5 feet 4 inches tall and scale the inserted people so they look naturally sized beside them. They should not appear giant, miniature, stretched, or squashed. Maintain natural human proportions with realistic head-to-body ratio and limb length so the group looks like real friends standing together or in what ever positon appears natural using the base canvas as your context in the same space. Keep all characters clearly visible within the 16:9 frame, avoiding crops that cut off heads or important limbs. Do not bring any part of the user's background into the final image. Ensure lighting and color of the inserted figures are harmonized with Image 2. Do not add any new objects, text, or logos. Avoid extra or duplicated limbs, distorted anatomy, or motion blur. The final image should look like a single, coherent Picasso-style scene suitable as a clean keyframe for AI video generation.
```

### Key Elements to Preserve

| Element | Detail |
|---------|--------|
| Scaling Reference | Picasso character ~5'4" tall, users scaled proportionally |
| Positioning | Close friends, facing each other, warm body language |
| Interaction Style | "Joyfully chumming around and having fun" |
| Identity | User remains recognizable, restyled to match Picasso aesthetic |
| Background | Use Picasso artwork as base, remove user's background |
| Proportions | Natural human proportions, realistic head-to-body ratio |
| Framing | 16:9, all characters visible, no cropped heads/limbs |
| Quality | No duplicated limbs, distorted anatomy, or motion blur |

---

## Extracted Prompt 2: Kie.ai Wan 2.5 (Video Generation)

**Node:** `Prepare Video Prompt`
**Purpose:** Animate the composed image into video

```
Animate this composed image showing people interacting with Picasso in a friendly, playful manner. Keep the artistic style consistent throughout the animation.
```

### Key Elements to Preserve

| Element | Detail |
|---------|--------|
| Interaction | Friendly, playful manner |
| Style Consistency | Artistic style maintained throughout |
| Subject Focus | People interacting with Picasso |

---

## Consolidation Analysis

### v2.4 Two-Stage Approach
```
Stage 1 (Reve Remix): Image composition with detailed positioning/scaling
Stage 2 (Kie.ai): Simple animation directive
```

### v3.0 Single-Stage Approach
```
Stage 1 (Kling O1): Must combine composition logic + animation directive
```

---

## Gap Analysis

| Reve Remix Detail | Current Kling Prompt | Status |
|-------------------|---------------------|--------|
| 5'4" scaling reference | Not included | MISSING |
| "Close friends" positioning | Partial ("warm, welcoming") | PARTIAL |
| Background removal | Not applicable (video) | N/A |
| Proportions specification | "Maintain stable identity" | PARTIAL |
| 16:9 framing | Included | OK |
| No duplicated limbs | Not included | MISSING |
| "Joyfully chumming around" | "Friendly banter and laughter" | PARTIAL |

---

## Recommendation

Update the Kling O1 prompt in `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` (Node 1: Prepare Kling Elements) to incorporate the missing details from the Reve Remix prompt.

---

## Related Documents

- `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` - Target configuration to update
- `docs/analysis/analysis-workflow-reconfiguration-plan-v1.0-2026-01-06.md` - Implementation plan
- `source/workflows/Picture with Picasso -Wan 2.5-v2.4.json` - Source workflow
