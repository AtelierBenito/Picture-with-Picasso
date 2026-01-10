# Kling O1 Video Generation Prompt

**Version:** v1.5-2026-01-09

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They embrace, laugh, and horse around with the warmth of a long-awaited reunion.

VISUAL FIDELITY - @Element2 IS THE MASTER:
All visual characteristics derived from @Element2: color palette (or sepia/B&W if source is), film grain, texture, lighting direction, shadow depth, color temperature. Preserve period-authentic environment exactly as shown - architecture, furnishings, surfaces, fixtures. No modern elements added.

IDENTITY PRESERVATION - CRITICAL:
All people remain EXACTLY as in their source images. Same face, same hairstyle, same hair color, same clothing. No modifications whatsoever. No aging or de-aging. No added facial hair on clean-shaven faces. No invented features for unseen parts. Visitors adopt @Element2's color/grain/texture but their physical appearance is unchanged.

CHARACTERS:
Picasso is fully animated and expressive, not a background figure. All characters move, gesture, react, and speak with equal energy and presence. Scale naturally: Picasso ~5'4" tall.

INTERACTION:
Reunion energy - excited greetings, warm embraces, playful banter. Picasso actively engages: laughing heartily, gesturing expressively, clapping friends on back, playful nudges. Affectionate teasing like old friends who know each other well. Surprised reactions, spontaneous joy, emotional warmth. The energy of friends who haven't seen each other in years.

AUDIO:
Expressive vocalizations - warm laughter, joyful exclamations, emotional sounds. Short impulsive expressions: "Hey!", "Yes!", "Ah, my friend!", cheers, delighted gasps. Picasso's voice has Spanish-accented warmth and energy. Ambient sounds authentic to @Element2 environment: if studio - creaking wood floors, rustling papers, muffled street sounds; if café - period glassware, French conversation murmur; if outdoor - period-appropriate street atmosphere. Lip movements match vocalizations naturally.

CAMERA:
Subtle naturalistic movement - gentle push toward subjects or slow drift around group. Intimate, personal feel that draws viewer into the moment.

TECHNICAL:
Maintain exact likeness throughout entire video. Natural human proportions, no duplicated limbs or distorted anatomy. Smooth cinematic motion. Realistic talking mouth movements with clear lip sync. 9:16 vertical aspect ratio. 10 second duration.
```

---

## Negative Prompt

```
face morphing, identity swap, identity change, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, different hairstyle, changed hair color, different hair length, changed clothing, different clothing, aged face, de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading, Instagram filter, contemporary color correction, converted color tone, digital smoothing, smoothed grain, removed grain, modern clarity, LED lighting, plastic fixtures, modern furniture, synthetic materials, contemporary architecture, modern glass, added environment elements, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, silent video, no audio, out of sync lip movements, mismatched audio, closed mouths while speaking, modern vehicles, changed lighting direction, electronic sounds, digital audio, contemporary music, modern background noise
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
| Main prompt | ~2,383 | 2,500 | OK |
| Negative prompt | ~1,050 | TBD | OK |

---

## Version Notes

**v1.5 Changes (2026-01-09):**

### Critical Fix
- **Reduced main prompt from ~5,600 to ~2,150 characters** (under 2,500 limit)
- Root cause of execution 3981 failure: `422 - ensure this value has at most 2500 characters`

### What Was Added Back (vs initial v1.5 draft)
1. **Enhanced INTERACTION section** - playful banter, affectionate teasing, surprised reactions, emotional warmth, "friends who haven't seen each other in years"
2. **Detailed ambient sounds** - studio (creaking floors, rustling papers, street sounds), café (glassware, French murmur), outdoor (period street atmosphere)
3. **CAMERA section** - gentle push, slow drift, intimate feel
4. **Additional audio details** - delighted gasps, joyful exclamations

### What Was Removed (from v1.4)
1. **DO NOT section** - Moved entirely to negative prompt (already there)
2. **Verbose if/then structures** - Condensed to single directives
3. **Separate PICASSO/VISITORS subsections** - Merged into single IDENTITY block

### What Was Preserved
1. Core narrative (reunion with Picasso)
2. @Element2 as visual master (color, tone, grain, lighting, environment)
3. Identity preservation (no face/hair/clothing changes)
4. No facial hair on clean-shaven faces
5. Audio generation with Spanish-accented Picasso
6. Period authenticity requirement
7. All technical specifications

---

## Key Principles (Unchanged)

1. **@Element2 = Visual master** for color, tone, grain, lighting, environment
2. **All people unchanged** from source images - no modifications
3. **Visitors placed INTO Picasso's visual world** - same color/tone/grain, identity unchanged
