# Kling O1 Video Prompt

**Version:** v1.9.2-2026-01-11
**Pairs with:** source-prompts-Kling_O1_Negative_Prompt-v1.9.2-2026_01_11.md

---

## Video Prompt

```
GROUP REUNION – THE HEART OF THIS VIDEO:
Everyone in this frame—@Element1 (visitors) and @Element2 (Picasso)—are great friends who haven't seen each other in years. This is the surprise reunion. They are ecstatic. They can't believe it.

ALL figures interact with EACH OTHER, not just with Picasso:
- Visitors embrace each other AND Picasso
- Group hugs, arms around multiple shoulders
- Friends grabbing friends, pulling them closer
- Shared laughter rippling through the group
- Playful shoving, back-patting, hand-clasping between everyone
- The energy of "I can't believe you're HERE!"

Physical contact between ALL parties is REQUIRED. No one stands apart. No polite distance. No observer figures. Everyone is IN the reunion.

IDENTITY – EXACT LIKENESS (NON-NEGOTIABLE):
Every person in @Element1 must be visually identical to their source appearance. Lock: face shape, facial features, skin tone, hair, clothing, accessories. Frame-to-frame consistency. No drift, no morphing, no interpretation. @Element2 (Picasso): Preserve exactly as shown—every wrinkle, expression, detail.

VISUAL FIDELITY – @Element2 IS THE MASTER:
Adopt @Element2's entire visual profile: color palette (or sepia/B&W), grain, texture, lighting direction, shadow depth. Preserve background, architecture, furnishings exactly. @Element1 inherits this treatment while keeping exact likeness.

SCALE:
Picasso is 5'4" (163cm). Size all others proportionally from source appearance.

MOTION:
Organic, joyful movement. Everyone actively animated. Picasso is fully present, never a background figure. The group moves as friends do—toward each other.

CAMERA:
Naturalistic—slow drift or gentle push toward the group. Cinematic feel.

TECHNICAL:
9:16 ratio. 10 seconds. Smooth motion throughout.
```

---

## Character Count

| Field | Count | Status |
|-------|-------|--------|
| Video prompt | ~1,480 | Full version |

---

## Condensed Version (under 1000 chars)

```
GROUP REUNION: @Element1 (visitors) and @Element2 (Picasso) are great friends reuniting after years apart. Ecstatic, surprised, joyful.

ALL figures interact with EACH OTHER—group hugs, arms around shoulders, playful shoving, back-patting, hand-clasping. Physical contact between ALL required. No one stands apart.

IDENTITY: Every @Element1 face exactly matches source—no drift, no morphing. @Element2 preserved exactly.

VISUAL: Adopt @Element2's color, grain, texture, lighting. @Element1 inherits this while keeping exact likeness.

SCALE: Picasso 5'4". Others proportional to source.

MOTION: Joyful, organic. Everyone animated. Group moves toward each other.

CAMERA: Slow drift toward group. TECHNICAL: 9:16, 10 seconds.
```

**Character count:** ~780 chars

---

## v1.9.2 Changes (from v1.9.1)

### Major Restructure
- Moved interaction to FIRST section (primacy for model attention)
- Changed from "old friends" to "great friends reuniting after years"
- Added "ecstatic" and "surprised" emotional anchors
- Emphasized ALL-to-ALL interaction (not just Picasso-to-visitor)

### Added
- "Group hugs, arms around multiple shoulders"
- "Friends grabbing friends, pulling them closer"
- "I can't believe you're HERE!" energy
- "No observer figures. Everyone is IN the reunion."
- Explicit list of group interaction types

### Removed
- "platonic" (per user request)
- "MOTION ORIGIN" section (was anchoring to static poses)
- "BEHAVIORAL VARIABILITY" section (replaced with group dynamics)

### Strengthened
- Identity section moved higher, language tightened
- "NON-NEGOTIABLE" added to identity directive
- Physical contact framed as REQUIRED not suggested

---

## Usage

Copy the prompt text (without markdown code fences) into the `prompt` field in the `Prepare Kling Elements1` node.

If character limit errors occur, use the condensed version.

---

## Interaction Randomization (Optional Enhancement)

Add a Code node before prompt assembly to inject specific group dynamics:

```javascript
const interactions = [
  "The whole group collapses into a spontaneous group hug, arms wrapping around everyone",
  "Friends grab each other by the shoulders in disbelief, laughing and pulling closer",
  "Everyone reaches for everyone—hands clasping, backs being patted, pure joy",
  "The group clusters together, arms around each other's shoulders, beaming",
  "Friends playfully shove each other while laughing, then pull into embraces",
  "Picasso pulls two visitors in while they reach for each other too—triangle of warmth",
  "The reunion erupts—hugging, hand-holding, joyful chaos between all friends",
  "Everyone leans in together, foreheads almost touching, sharing the moment"
];

const selected = interactions[Math.floor(Math.random() * interactions.length)];
return [{ json: { interaction: selected } }];
```

Then inject `{{ $json.interaction }}` after "I can't believe you're HERE!" line.
