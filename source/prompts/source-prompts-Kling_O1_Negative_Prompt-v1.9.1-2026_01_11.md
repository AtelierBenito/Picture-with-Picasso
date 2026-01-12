# Kling O1 Negative Prompt

**Version:** v1.9.1-2026-01-11
**Pairs with:** source-prompts-Kling_O1_Video_Prompt-v1.9.1-2026_01_11.md

---

## Negative Prompt

```
face morphing, identity swap, identity change, identity drift, identity mixing, blended identities, face morphing between frames, changing features, approximate likeness, blended features, interpreted likeness, AI interpretation of face, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, changed hairstyle, changed hair color, changed clothing, aged face, de-aged face, invented features, modern color grading, Instagram filter, digital smoothing, digital clarity, modern overlays, removed grain, modern clarity, LED lighting, plastic fixtures, modern furniture, contemporary architecture, modern glass, modern vehicles, added elements, stretched figures, shrunk figures, visual tricks, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, repeated poses, identical gestures, looping motion, mechanical movement, abrupt motion, jerky movement, wrong height, wrong proportions
```

---

## Character Count

| Field | Count | Status |
|-------|-------|--------|
| Negative prompt | ~980 | Full version |

---

## Condensed Version (under 500 chars)

```
face morphing, identity drift, identity mixing, blended features, approximate likeness, AI interpretation, changed facial hair, changed hairstyle, changed clothing, aged face, invented features, modern elements, Instagram filter, digital smoothing, added elements, stretched figures, duplicated limbs, distorted anatomy, static Picasso, frozen characters, stiff poses, repeated poses, abrupt motion, jerky movement, wrong proportions
```

**Character count:** ~450 chars

---

## v1.9.1 Changes (from v1.9)

### Added (gaps from main prompt cleanup)
- identity mixing
- blended identities
- blended features
- interpreted likeness
- digital clarity
- modern overlays
- added elements
- stretched figures
- shrunk figures
- visual tricks
- abrupt motion
- jerky movement

### Kept from v1.9
- All identity preservation terms
- All period authenticity terms
- All variability terms
- Height/proportion terms

---

## Usage

Copy the negative prompt text (without markdown code fences) into the `negative_prompt` field in the `Prepare Kling Elements1` node.

If character limit errors occur, use the condensed version.
