# Kling O1 Video Generation Prompt

**Version:** v1.8-2026-01-10

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They embrace, laugh, and horse around with the warmth of a long-awaited reunion.

VISUAL FIDELITY - @Element2 IS THE MASTER:
All visual characteristics from @Element2: color palette (or sepia/B&W if source is), film grain, texture, lighting, shadow depth. Preserve period-authentic environment exactly - architecture, furnishings, fixtures. No modern elements.

IDENTITY - PIXEL-PERFECT ACCURACY:
Every person matches their source image EXACTLY - pixel-level accuracy for all facial features, skin texture, hair, and clothing. No approximations, no AI interpretations.

@Element2 (Picasso): Reproduce his face exactly as shown. Every feature, every detail, frame-by-frame consistency. If clean-shaven in source, he remains clean-shaven. No drift, no morphing.

@Element1 (Visitors): Reproduce each face exactly as shown. Same eyes, nose, mouth, skin, hair. Lock identity from first to last frame. No blending between people, no identity drift.

ONLY adaptation: apply @Element2's color/grain/texture to visitors so they exist in the same visual world. Physical identity remains pixel-perfect.

SCALE: Picasso is 5'4" (163cm). Size all others proportionally. Taller visitors should tower over Picasso.

CHARACTERS: Picasso fully animated and expressive. All characters move, gesture, react, and speak with equal energy.

INTERACTION: Reunion energy - excited greetings, warm embraces, playful banter. Picasso actively engages: laughing, gesturing, clapping friends on back.

AUDIO REQUIRED - THIS VIDEO MUST HAVE SOUND:
- Picasso's voice: warm, energetic, Spanish-accented male voice
- Vocalizations: "Hey!", "Ah, my friend!", joyful exclamations, hearty laughter
- Visitor voices: natural responses, laughter
- Ambient audio: period-authentic sounds from @Element2 environment
- Lip sync: mouth movements must match audio

DO NOT generate a silent video. Audio is mandatory.

CAMERA: Subtle naturalistic movement - gentle push toward subjects or slow drift around group.

TECHNICAL: Exact likeness throughout. Natural proportions, no duplicated limbs. Smooth motion. Lip sync. 9:16 ratio. 10 seconds.
```

---

## Negative Prompt

```
face morphing, identity swap, identity change, identity drift, face morphing between frames, changing features, approximate likeness, AI interpretation of face, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, changed hair color, changed clothing, aged face, de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading, digital smoothing, smoothed grain, removed grain, modern clarity, LED lighting, modern furniture, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, silent video, no audio, muted, no sound, soundless, out of sync lip movements, mismatched audio, closed mouths while speaking, Picasso taller than 5'4", wrong height, wrong proportions
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
| Main prompt | ~2,405 | 2,500 | OK (~95 buffer) |
| Negative prompt | ~883 | TBD | OK |

---

## Version Notes

**v1.8 Changes (2026-01-10):**

### Critical Fix
- **Reduced from ~2,458 (v1.7) to ~2,405 characters** - added ~50 char buffer
- Execution 3987 failed with: `prompt: size must be between 0 and 2500`
- Root cause: JSON double-escaping inflated prompt to ~2,538 chars

### What Was Trimmed (vs v1.7)
1. Removed "no artistic liberties" (covered by "no AI interpretations")
2. Removed "no gradual changes across frames" (covered by "no drift, no morphing")
3. Removed "their" from "Lock their identity"
4. Removed "to source" from "Physical identity remains pixel-perfect"
5. Shortened "SCALE CRITICAL" to "SCALE"
6. Removed "Never make Picasso appear taller than 5'4"" (redundant)
7. Removed "not a background figure" from CHARACTERS
8. Removed "heartily" and "expressively" from INTERACTION
9. Removed "playful nudges" from INTERACTION
10. Removed "Intimate, personal feel" from CAMERA

### Key Content Preserved
1. Full visual fidelity section
2. Pixel-perfect identity requirements
3. Frame-by-frame consistency
4. Complete audio section with all requirements
5. Height/scale requirements
6. All technical specifications

---

## Key Principles

1. **@Element2 = Visual master** for color, tone, grain, lighting, environment
2. **Pixel-perfect identity** - no approximations, no drift, no AI interpretation
3. **Picasso = 5'4"** - proportional scaling for all characters
4. **Audio is mandatory** - explicit requirement for sound generation
