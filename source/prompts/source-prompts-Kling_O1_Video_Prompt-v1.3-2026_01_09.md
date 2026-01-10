# Kling O1 Video Generation Prompt

**Version:** v1.3-2026-01-09

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are old friends reuniting after a long time apart. They are overjoyed to see each other - embracing, laughing, and horsing around together with the warmth and energy of a long-awaited reunion.

CHARACTERS:
- Picasso is fully animated and expressive - not a background figure
- Both visitors AND Picasso move, gesture, react, and SPEAK to each other
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
- Spontaneous expressions of joy - surprised reactions, warm embraces, playful nudges
- The energy of friends who haven't seen each other in a long time

AUDIO:
- Warm, expressive vocalizations - laughter, exclamations, emotional sounds
- Short impulsive expressions: "Hey!", "Yes!", "Ah, my friend!", cheers, hoots
- Picasso's voice has Spanish-accented warmth and energy
- Happy sounds during embraces - sighs of joy, delighted gasps
- Playful teasing sounds - affectionate "talented scoundrel!" type exclamations
- Ambient sounds authentic to the scene (café chatter, studio sounds, outdoor atmosphere)
- Lip movements match vocalizations naturally

CAMERA:
- Subtle, naturalistic movement that feels personal and intimate
- Gentle push in toward the subjects, or slow drift around the group
- Movement that draws viewer into the moment

TECHNICAL:
- Maintain exact likeness of all people throughout
- Natural human proportions, no duplicated limbs or distorted anatomy
- Smooth, cinematic motion
- Realistic talking mouth movements with clear lip sync
- 9:16 aspect ratio (vertical, social media native)
- 10 second duration
```

---

## Negative Prompt

```
face morphing, identity swap, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur, static Picasso, frozen characters, stiff poses, silent video, no audio, out of sync lip movements, mismatched audio, closed mouths while speaking
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

**v1.3 Changes:**
- Added dedicated AUDIO section optimized for Kling-Foley capabilities
- Focus on expressive vocalizations (laughter, exclamations, emotional sounds) vs full dialogue
- Short impulsive expressions: "Hey!", "Yes!", "Ah, my friend!", cheers, hoots
- Picasso's voice: Spanish-accented warmth and energy
- Happy sounds during embraces, playful teasing sounds
- Simplified INTERACTION to focus on spontaneous joy vs scripted dialogue
- Added "and SPEAK" to CHARACTERS section
- Added lip sync direction for vocalizations
- Added to TECHNICAL: "Realistic talking mouth movements with clear lip sync"
- Added to negative prompt: lip sync and audio quality controls
- Ambient sounds scene-adaptive (café, studio, outdoor)
