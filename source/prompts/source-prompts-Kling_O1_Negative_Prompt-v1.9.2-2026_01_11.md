# Kling O1 Negative Prompt

**Version:** v1.9.2-2026-01-11
**Pairs with:** source-prompts-Kling_O1_Video_Prompt-v1.9.2-2026_01_11.md

---

## Negative Prompt

```
standing apart, isolated figures, polite distance, observer posture, watching from sidelines, one-on-one only, ignoring other visitors, separate conversations, formal arrangement, posed group photo, stiff lineup, generic hand gestures, hands raised without purpose, waving at camera, arms at sides, no physical contact, distant body language, formal posture, face morphing, identity drift, identity swap, identity mixing, blended features, approximate likeness, AI interpretation of face, changed facial hair, changed hairstyle, changed clothing, aged face, invented features, modern elements, modern color grading, Instagram filter, digital smoothing, static Picasso, frozen characters, static figures, stiff poses, repeated poses, identical gestures, looping motion, mechanical movement, abrupt motion, jerky movement, wrong height, wrong proportions, duplicated limbs, distorted anatomy
```

---

## Character Count

| Field | Count | Status |
|-------|-------|--------|
| Negative prompt | ~920 | Full version |

---

## Condensed Version (under 500 chars)

```
standing apart, isolated figures, polite distance, observer posture, one-on-one only, formal arrangement, posed group photo, generic hand gestures, waving at camera, no physical contact, face morphing, identity drift, blended features, approximate likeness, changed appearance, modern elements, Instagram filter, static figures, frozen characters, stiff poses, repeated poses, jerky movement, wrong proportions
```

**Character count:** ~420 chars

---

## v1.9.2 Changes (from v1.9.1)

### Added (Group Dynamics Blocking)
- standing apart
- isolated figures
- polite distance
- observer posture
- watching from sidelines
- one-on-one only
- ignoring other visitors
- separate conversations
- formal arrangement
- posed group photo
- stiff lineup
- hands raised without purpose
- distant body language
- formal posture
- static figures

### Kept from v1.9.1
- All identity preservation terms (face morphing, identity drift, etc.)
- All period authenticity terms (modern elements, Instagram filter, etc.)
- All motion quality terms (jerky movement, stiff poses, etc.)
- Anatomy terms (duplicated limbs, distorted anatomy, etc.)

### Removed
- Some redundant terms consolidated

---

## Usage

Copy the negative prompt text (without markdown code fences) into the `negative_prompt` field in the `Prepare Kling Elements1` node.

If character limit errors occur, use the condensed version.

---

## Category Breakdown

| Category | Terms |
|----------|-------|
| **Anti-Isolation** | standing apart, isolated figures, polite distance, observer posture, watching from sidelines |
| **Anti-Generic** | one-on-one only, formal arrangement, posed group photo, stiff lineup, generic hand gestures |
| **Anti-Passive** | waving at camera, arms at sides, no physical contact, distant body language |
| **Identity Lock** | face morphing, identity drift, identity swap, identity mixing, blended features, approximate likeness |
| **Appearance Lock** | changed facial hair, changed hairstyle, changed clothing, aged face, invented features |
| **Period Authenticity** | modern elements, modern color grading, Instagram filter, digital smoothing |
| **Motion Quality** | static Picasso, frozen characters, stiff poses, repeated poses, jerky movement |
| **Anatomy** | duplicated limbs, distorted anatomy, wrong height, wrong proportions |
