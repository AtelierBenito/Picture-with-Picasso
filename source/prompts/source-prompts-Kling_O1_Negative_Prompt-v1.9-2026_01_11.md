# Kling O1 Negative Prompt

**Version:** v1.9-2026-01-11
**Pairs with:** source-prompts-Kling_O1_Video_Prompt-v1.9-2026_01_11.md

---

## Negative Prompt

```
face morphing, identity swap, identity change, identity drift, face morphing between frames, changing features, approximate likeness, AI interpretation of face, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, changed hair color, changed clothing, aged face, de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading, Instagram filter, digital smoothing, smoothed grain, removed grain, modern clarity, LED lighting, plastic fixtures, modern furniture, contemporary architecture, modern glass, modern vehicles, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, repeated poses, identical gestures, looping motion, mechanical movement, Picasso taller than 5'4", wrong height, wrong proportions
```

---

## Character Count

| Field | Count | Limit | Status |
|-------|-------|-------|--------|
| Negative prompt | ~920 | 500 (fal.ai) | Review needed |

**Note:** fal.ai may have a 500-character limit on negative prompts. If errors occur, use the condensed version below.

---

## Condensed Version (if needed)

```
face morphing, identity drift, identity swap, changing features, approximate likeness, AI interpretation, added facial hair, removed facial hair, changed hairstyle, changed hair color, changed clothing, aged face, de-aged face, invented features, modern elements, Instagram filter, digital smoothing, LED lighting, contemporary architecture, modern vehicles, duplicated limbs, distorted anatomy, motion blur, static Picasso, frozen characters, stiff poses, repeated poses, looping motion, wrong height, wrong proportions
```

**Character count:** ~495 chars

---

## v1.9 Changes

### Removed (audio not supported)
- silent video
- no audio
- muted
- no sound
- soundless
- out of sync lip movements
- mismatched audio
- closed mouths while speaking

### Added back from v1.5 (period authenticity)
- Instagram filter
- plastic fixtures
- contemporary architecture
- modern glass
- modern vehicles

### Added new (behavioral variability)
- repeated poses
- identical gestures
- looping motion
- mechanical movement

### Kept from v1.8 (identity preservation)
- identity drift
- face morphing between frames
- changing features
- approximate likeness
- AI interpretation of face
- Picasso taller than 5'4"
- wrong height
- wrong proportions

---

## Usage in n8n

Copy the negative prompt text (without markdown code fences) into the `negative_prompt` field in the `Prepare Kling Elements1` node.

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.9 | 2026-01-11 | Removed audio blockers, added period/variability blockers |
| v1.8 | 2026-01-10 | Added identity drift, height constraints |
| v1.5 | 2026-01-09 | Initial comprehensive negative prompt |
