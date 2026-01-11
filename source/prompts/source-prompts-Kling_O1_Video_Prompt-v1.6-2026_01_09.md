# Kling O1 Video Generation Prompt

**Version:** v1.6-2026-01-09

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They embrace, laugh, and horse around with the warmth of a long-awaited reunion.

VISUAL FIDELITY - @Element2 IS THE MASTER:
All visual characteristics derived from @Element2: color palette (or sepia/B&W if source is), film grain, texture, lighting direction, shadow depth, color temperature. Preserve period-authentic environment exactly as shown - architecture, furnishings, surfaces, fixtures. No modern elements added.

IDENTITY PRESERVATION - PIXEL-PERFECT ACCURACY:
Every person must match their source image EXACTLY - pixel-level accuracy for all facial features, skin texture, hair, and clothing. No approximations, no AI interpretations, no artistic liberties.

@Element2 (Picasso): Reproduce his face exactly as shown. Every feature, every detail, frame-by-frame consistency. If clean-shaven in source, he remains clean-shaven. No drift, no morphing, no gradual changes across frames.

@Element1 (Visitors): Reproduce each face exactly as shown. Same eyes, same nose, same mouth, same skin, same hair. Lock their identity from first frame to last. No blending between people, no feature averaging, no identity drift.

The ONLY adaptation: apply @Element2's color/grain/texture to visitors so they exist in the same visual world. Their physical identity remains pixel-perfect to source.

SCALE CRITICAL:
Picasso is 5'4" (163cm) tall. All other characters must be sized proportionally relative to Picasso's height. If visitors are taller, they should visibly tower over Picasso. If similar height, their eye levels should align. Never make Picasso appear taller than 5'4".

CHARACTERS:
Picasso is fully animated and expressive, not a background figure. All characters move, gesture, react, and speak with equal energy and presence.

INTERACTION:
Reunion energy - excited greetings, warm embraces, playful banter. Picasso actively engages: laughing heartily, gesturing expressively, clapping friends on back, playful nudges. Affectionate teasing like old friends who know each other well.

AUDIO GENERATION REQUIRED - THIS VIDEO MUST HAVE SOUND:
Generate audio track synchronized to the video. Include:
- Picasso's voice: warm, energetic, Spanish-accented male voice
- Visitor voices: natural responses, laughter
- Vocalizations: "Hey!", "Ah, my friend!", joyful exclamations, hearty laughter
- Ambient audio: period-authentic sounds from @Element2 environment
- Lip sync: mouth movements must match audio

DO NOT generate a silent video. Audio is mandatory.

CAMERA:
Subtle naturalistic movement - gentle push toward subjects or slow drift around group. Intimate, personal feel.

TECHNICAL:
Maintain exact likeness throughout entire video. Natural human proportions, no duplicated limbs. Smooth cinematic motion. Realistic talking mouth movements with clear lip sync. 9:16 vertical aspect ratio. 10 second duration.
```

---

## Negative Prompt

```
face morphing, identity swap, identity change, identity drift, face morphing between frames, changing features, approximate likeness, AI interpretation of face, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, different hairstyle, changed hair color, different hair length, changed clothing, different clothing, aged face, de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading, Instagram filter, contemporary color correction, converted color tone, digital smoothing, smoothed grain, removed grain, modern clarity, LED lighting, plastic fixtures, modern furniture, synthetic materials, contemporary architecture, modern glass, added environment elements, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, silent video, no audio, muted, no sound, soundless, out of sync lip movements, mismatched audio, closed mouths while speaking, modern vehicles, changed lighting direction, electronic sounds, digital audio, contemporary music, modern background noise, Picasso taller than 5'4", wrong height, wrong proportions
```

---

## Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| `duration` | `10` | |
| `aspect_ratio` | `9:16` | |
| `generate_audio` | `true` | **CRITICAL: Must be included** |

---

## API Request Body

```javascript
={{ JSON.stringify({
  "prompt": $json.prompt,
  "negative_prompt": $json.negative_prompt,
  "elements": $json.elements,
  "duration": $json.duration,
  "aspect_ratio": $json.aspect_ratio,
  "generate_audio": true
}) }}
```

---

## Character Count

| Field | Count | Limit | Status |
|-------|-------|-------|--------|
| Main prompt | ~2,180 | 2,500 | OK |
| Negative prompt | ~1,150 | TBD | OK |

---

## Version Notes

**v1.6 Changes (2026-01-09):**

### Height Reinforcement (NEW)
- Added dedicated **SCALE CRITICAL** section
- Explicit height: "Picasso is 5'4" (163cm) tall"
- Proportional sizing instruction for all characters
- "Never make Picasso appear taller than 5'4""

### Pixel-Perfect Identity (ENHANCED)
- Replaced generic identity section with **PIXEL-PERFECT ACCURACY** directive
- "pixel-level accuracy for all facial features, skin texture, hair, and clothing"
- "No approximations, no AI interpretations, no artistic liberties"
- Frame-by-frame consistency requirement
- "Lock their identity from first frame to last"
- "No blending between people, no feature averaging, no identity drift"

### Audio Reinforcement (ENHANCED)
- New explicit section: **AUDIO GENERATION REQUIRED - THIS VIDEO MUST HAVE SOUND**
- Bullet list of required audio elements
- Explicit statement: "DO NOT generate a silent video. Audio is mandatory."

### Negative Prompt (ENHANCED)
Added:
- `identity drift`, `face morphing between frames`, `changing features`
- `approximate likeness`, `AI interpretation of face`
- `silent video`, `muted`, `no sound`, `soundless`
- `Picasso taller than 5'4"`, `wrong height`, `wrong proportions`

### What Was Preserved
1. Core narrative (reunion with Picasso)
2. @Element2 as visual master
3. Period authenticity requirement
4. All technical specifications
5. Camera movement instructions

---

## Key Principles

1. **@Element2 = Visual master** for color, tone, grain, lighting, environment
2. **Pixel-perfect identity** - no approximations, no drift, no AI interpretation
3. **Picasso = 5'4"** - proportional scaling for all characters
4. **Audio is mandatory** - explicit requirement for sound generation
