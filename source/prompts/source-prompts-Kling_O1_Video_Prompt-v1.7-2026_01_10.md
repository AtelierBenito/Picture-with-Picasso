# Kling O1 Video Generation Prompt

**Version:** v1.7-2026-01-10

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They embrace, laugh, and horse around with the warmth of a long-awaited reunion.

VISUAL FIDELITY - @Element2 IS THE MASTER:
All visual characteristics from @Element2: color palette (or sepia/B&W if source is), film grain, texture, lighting, shadow depth. Preserve period-authentic environment exactly - architecture, furnishings, fixtures. No modern elements.

IDENTITY - PIXEL-PERFECT ACCURACY:
Every person matches their source image EXACTLY - pixel-level accuracy for all facial features, skin texture, hair, and clothing. No approximations, no AI interpretations, no artistic liberties.

@Element2 (Picasso): Reproduce his face exactly as shown. Every feature, every detail, frame-by-frame consistency. If clean-shaven in source, he remains clean-shaven. No drift, no morphing, no gradual changes across frames.

@Element1 (Visitors): Reproduce each face exactly as shown. Same eyes, nose, mouth, skin, hair. Lock their identity from first frame to last. No blending between people, no feature averaging, no identity drift.

ONLY adaptation: apply @Element2's color/grain/texture to visitors so they exist in the same visual world. Physical identity remains pixel-perfect to source.

SCALE CRITICAL: Picasso is 5'4" (163cm) tall. Size all others proportionally. Taller visitors should visibly tower over Picasso. Never make Picasso appear taller than 5'4".

CHARACTERS: Picasso fully animated and expressive, not a background figure. All characters move, gesture, react, and speak with equal energy and presence.

INTERACTION: Reunion energy - excited greetings, warm embraces, playful banter. Picasso actively engages: laughing heartily, gesturing expressively, clapping friends on back, playful nudges.

AUDIO REQUIRED - THIS VIDEO MUST HAVE SOUND:
- Picasso's voice: warm, energetic, Spanish-accented male voice
- Vocalizations: "Hey!", "Ah, my friend!", joyful exclamations, hearty laughter
- Visitor voices: natural responses, laughter
- Ambient audio: period-authentic sounds from @Element2 environment
- Lip sync: mouth movements must match audio

DO NOT generate a silent video. Audio is mandatory.

CAMERA: Subtle naturalistic movement - gentle push toward subjects or slow drift around group. Intimate, personal feel.

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
| Main prompt | ~2,350 | 2,500 | OK |
| Negative prompt | ~950 | TBD | OK |

---

## Version Notes

**v1.7 Changes (2026-01-10):**

### Critical Fix
- **Reduced from ~2,920 (v1.6) to ~2,350 characters** - under 2,500 limit with 150 buffer
- Execution 3986 failed with: `422 - ensure this value has at most 2500 characters`

### What v1.7 Includes (Restored from v1.6)
1. Full visual fidelity details (sepia/B&W option, architecture, furnishings, surfaces, fixtures)
2. "Every feature, every detail" identity language
3. Full pixel-perfect accuracy directive
4. Complete audio section with all bullet points
5. "Intimate, personal feel" for camera
6. Height reinforcement with 5'4" (163cm) and proportional sizing
7. Frame-by-frame consistency requirement

### What Was Trimmed (vs v1.6)
1. Removed some redundant phrasing
2. Condensed TECHNICAL section slightly
3. Removed duplicate concepts covered in negative prompt

### Key Improvements Over v1.5
1. **PIXEL-PERFECT ACCURACY** - stronger identity directive
2. **SCALE CRITICAL** - explicit 5'4" height with proportional sizing
3. **AUDIO REQUIRED** - explicit "DO NOT generate silent video" statement
4. Enhanced negative prompt with identity drift, wrong height terms

---

## Key Principles

1. **@Element2 = Visual master** for color, tone, grain, lighting, environment
2. **Pixel-perfect identity** - no approximations, no drift, no AI interpretation
3. **Picasso = 5'4"** - proportional scaling for all characters
4. **Audio is mandatory** - explicit requirement for sound generation
