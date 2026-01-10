# Kling O1 Video Generation Prompt

**Version:** v1.2-2026-01-09

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They are overjoyed to see each other - embracing, laughing, and horsing around together with the warmth and energy of a long-awaited reunion.

CHARACTERS:
- Picasso is fully animated and expressive - not a background figure
- Both visitors AND Picasso move, gesture, and react to each other
- Equal energy and presence from all characters

VISUAL STYLE:
- Tone, coloring, texture, and focus derived from @Element2 reference image
- Lighting matches the source image - warmth, direction, mood
- Environment authentic to the context shown in the Picasso reference

POSITIONING:
- Close together, naturally placed within the scene
- Facing toward each other with warm, connected body language
- All characters clearly visible within frame

INTERACTION:
- Reunion energy - excited greetings, happy embraces, playful banter
- Picasso actively engages: laughing, gesturing expressively, clapping friends on the back
- Affectionate teasing like old friends who know each other well
- Natural conversation sounds, laughter, ambient environment

CAMERA:
- Subtle, naturalistic movement that feels personal and intimate
- Gentle push in toward the subjects, or slow drift around the group
- Movement that draws viewer into the moment

TECHNICAL:
- Maintain exact likeness of all people throughout
- Natural human proportions, no duplicated limbs or distorted anatomy
- Smooth, cinematic motion
- 9:16 aspect ratio (vertical, social media native)
- 10 second duration
```

---

## Negative Prompt

```
face morphing, identity swap, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses
```

---

## Parameters

| Parameter | Value |
|-----------|-------|
| `duration` | `10` |
| `aspect_ratio` | `9:16` |
| `generate_audio` | `true` |

---

## Version Notes

**v1.2 Changes:**
- Reframed narrative: old friends reuniting after long time apart
- Made Picasso an actively animated character (not just style reference)
- Added "CHARACTERS" section emphasizing equal energy from all
- Enhanced interaction language: embracing, horsing around, clapping on back
- Added to negative prompt: static Picasso, frozen characters, stiff poses
