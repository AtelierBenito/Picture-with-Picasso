# Kling O1 Video Generation Prompt

**Version:** v1.4-2026-01-09

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They are overjoyed to see each other - embracing, laughing, and horsing around together with the warmth and energy of a long-awaited reunion.

== VISUAL MASTER: @Element2 IS THE SOURCE OF TRUTH ==

The @Element2 (Picasso) reference image defines ALL visual characteristics for the entire video:

COLOR & TONE:
- If @Element2 is COLOR → video is color, matching that exact color palette and saturation
- If @Element2 is SEPIA → video is sepia with that same tonal range
- If @Element2 is BLACK & WHITE → video is black and white
- Match the EXACT color temperature, warmth, and tonal qualities of @Element2

TEXTURE & GRAIN:
- If @Element2 has FILM GRAIN → video retains that grain texture throughout
- If @Element2 is GRAINY → video is grainy
- If @Element2 has SOFT FOCUS → video has soft focus
- PRESERVE the photographic texture - do NOT smooth to modern digital clarity

LIGHTING:
- Match the EXACT lighting direction from @Element2
- Match the EXACT shadow depth and contrast
- Match the light source quality (hard/soft, warm/cool)
- The lighting in @Element2 is authentic to its era - preserve it

ENVIRONMENT:
- The background/environment in @Element2 is AUTHENTIC to the period
- Preserve the architecture, furnishings, surfaces, and fixtures exactly as shown
- If showing Bateau-Lavoir studio: aged wood, bare plaster, period windows, sparse furnishings
- If showing café: period tables, vintage fixtures, authentic architecture
- DO NOT add any elements not present in @Element2

== ABSOLUTE IDENTITY PRESERVATION (CRITICAL) ==

ALL PEOPLE MUST REMAIN EXACTLY AS THEY APPEAR IN THEIR SOURCE IMAGES:

PICASSO (@Element2):
- Face must match @Element2 EXACTLY - no modifications whatsoever
- If clean-shaven in @Element2 → remains clean-shaven (NO mustache, NO beard, NO goatee)
- If has facial hair in @Element2 → preserve it exactly as shown
- Same hairstyle, same hairline, same facial structure
- Same clothing as shown in @Element2
- Do NOT age or de-age

VISITORS (@Element1):
- Faces must match @Element1 EXACTLY - no modifications whatsoever
- Same hairstyle as shown in @Element1 - do NOT change it
- Same hair color, same hair length, same hair configuration
- Same clothing as shown in @Element1
- Same facial features - no adding, removing, or changing anything
- Do NOT age or de-age
- Do NOT invent features for parts not visible (if back of head not visible, leave undefined)

The only visual adaptation: apply @Element2's color tone, grain, and texture to the visitors so they appear to exist in the same photographic world. Their actual physical appearance remains unchanged.

== CHARACTERS ==

- Picasso is fully animated and expressive - not a background figure
- Both visitors AND Picasso move, gesture, react, and SPEAK to each other
- Equal energy and presence from all characters
- Scale visitors naturally: Picasso was approximately 5'4" tall; size others proportionally

== POSITIONING ==

- Close together, naturally placed within the scene
- Facing toward each other with warm, connected body language
- All characters clearly visible within frame
- Natural human proportions, realistic head-to-body ratio

== INTERACTION ==

- Reunion energy - excited greetings, happy embraces, playful banter
- Picasso actively engages: laughing, gesturing expressively, clapping friends on the back
- Affectionate teasing like old friends who know each other well
- Spontaneous expressions of joy - surprised reactions, warm embraces, playful nudges

== AUDIO ==

- Warm, expressive vocalizations - laughter, exclamations, emotional sounds
- Short impulsive expressions: "Hey!", "Yes!", "Ah, my friend!", cheers, hoots
- Picasso's voice has Spanish-accented warmth and energy
- Happy sounds during embraces - sighs of joy, delighted gasps
- Playful teasing sounds - affectionate exclamations
- Ambient sounds AUTHENTIC to the environment shown in @Element2:
  - Studio: creaking wood floors, rustling papers, outdoor street sounds through windows
  - Café: period glassware, French conversation murmur, no electronic sounds
  - Outdoor: period-appropriate street sounds, no modern vehicles
- Lip movements match vocalizations naturally

== CAMERA ==

- Subtle, naturalistic movement that feels personal and intimate
- Gentle push in toward the subjects, or slow drift around the group
- Movement that draws viewer into the moment

== TECHNICAL ==

- Maintain exact likeness of all people throughout entire video
- Natural human proportions, no duplicated limbs or distorted anatomy
- Smooth, cinematic motion
- Realistic talking mouth movements with clear lip sync
- 9:16 aspect ratio (vertical, social media native)
- 10 second duration

== DO NOT (EXPLICIT PROHIBITIONS) ==

IDENTITY:
- DO NOT add facial hair to faces that are clean-shaven in source images
- DO NOT remove facial hair that exists in source images
- DO NOT change anyone's hairstyle from their source image
- DO NOT change anyone's hair color or length
- DO NOT change anyone's clothing
- DO NOT age or de-age anyone
- DO NOT add wrinkles, age spots, or features not in source images
- DO NOT invent features for unseen parts of anyone

VISUAL:
- DO NOT apply modern color grading or filters
- DO NOT convert the color/tone to something different from @Element2
- DO NOT add modern fixtures (LED lights, plastic, contemporary furniture)
- DO NOT smooth out film grain or vintage texture
- DO NOT change the lighting direction or quality from @Element2
- DO NOT add objects not present in the @Element2 environment
```

---

## Negative Prompt

```
face morphing, identity swap, identity change, added facial hair, removed facial hair, mustache on clean-shaven face, beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, different hairstyle, changed hair color, different hair length, changed clothing, different clothing, aged face, de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading, Instagram filter, contemporary color correction, converted color tone, digital smoothing, smoothed grain, removed grain, modern clarity, LED lighting, plastic fixtures, modern furniture, synthetic materials, contemporary architecture, modern glass, added environment elements, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, silent video, no audio, out of sync lip movements, mismatched audio, closed mouths while speaking, modern vehicles, changed lighting direction
```

---

## Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| `duration` | `10` | |
| `aspect_ratio` | `9:16` | |
| `generate_audio` | `true` | **CRITICAL: Must be included** |

---

## API Request Body (Required)

```json
{
  "prompt": "[PROMPT ABOVE]",
  "negative_prompt": "[NEGATIVE PROMPT ABOVE]",
  "elements": [
    {
      "reference_image_urls": ["[USER_IMAGE_BASE64]"],
      "frontal_image_url": "[USER_IMAGE_BASE64]"
    },
    {
      "reference_image_urls": ["[PICASSO_IMAGE_BASE64]"],
      "frontal_image_url": "[PICASSO_IMAGE_BASE64]"
    }
  ],
  "duration": "10",
  "aspect_ratio": "9:16",
  "generate_audio": true
}
```

---

## Version Notes

**v1.4 Changes (2026-01-09):**

### Critical Fixes
1. **Added `generate_audio: true`** - Root cause of missing audio in v1.3

### Visual Master Section (Restructured)
2. **@Element2 is the source of truth** - Picasso image defines ALL visual characteristics
3. **COLOR & TONE** block - Video inherits exact color/sepia/B&W from source
4. **TEXTURE & GRAIN** block - Video inherits grain/texture, NOT smoothed
5. **LIGHTING** block - Match exact direction, depth, quality
6. **ENVIRONMENT** block - Preserve authentic period environment exactly

### Absolute Identity Preservation (NEW - Critical)
7. **PICASSO section** - No modifications whatsoever to face, hair, clothing
8. **VISITORS section** - No modifications whatsoever to face, hair, clothing
9. Explicit: "Do NOT age or de-age" for both
10. Explicit: "Do NOT change hairstyle" for both
11. Explicit: "Do NOT change clothing" for both
12. Only adaptation allowed: apply @Element2's color/tone/grain to visitors

### DO NOT Section (Expanded)
13. Split into IDENTITY and VISUAL categories
14. 8 identity prohibitions + 6 visual prohibitions

### Negative Prompt (Enhanced)
15. Added identity-specific negatives: `identity change`, `changed hairstyle`, `different hairstyle`, `changed hair color`, `different hair length`, `changed clothing`, `different clothing`, `aged face`, `de-aged face`, `added wrinkles`, `removed wrinkles`

---

## Key Principles

1. **@Element2 (Picasso image) = Visual master** for color, tone, grain, lighting, environment
2. **All people remain exactly as photographed** - no changes to face, hair, clothing, age
3. **Visitors are placed INTO Picasso's visual world** - same color/tone/grain applied, but their identity unchanged
