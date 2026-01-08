# Kling O1 Video Generation Prompt

**Version:** v1.0-2026-01-08
**Target API:** fal-ai/kling-video/o1/reference-to-video
**Workflow:** Picture with Picasso v3.0

---

## Prompt

```
@Element1 (visitors) and @Element2 (Picasso) are close friends joyfully chumming around together.

VISUAL STYLE:
- Tone, coloring, texture, and focus derived from @Element2 reference image
- Lighting matches the source image - warmth, direction, mood
- Environment authentic to the context shown in the Picasso reference

POSITIONING:
- Positioned near each other, naturally placed within the scene
- Comfortable, friendly interaction and body language
- All characters clearly visible within frame

INTERACTION:
- Genuine friendship energy - animated conversation, shared laughter
- Affectionately calling each other "talented scoundrels" or other endearing, playful banter of Picasso's era
- Expressive gestures, warm eye contact, natural reactions
- Ambient sounds of the environment

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

## Configuration Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| `duration` | `10` | 10 seconds for meaningful interaction |
| `aspect_ratio` | `9:16` | Vertical, native to TikTok/Instagram Reels |

---

## Element Mapping

| Element | Source | Purpose |
|---------|--------|---------|
| `@Element1` | User photo | Visitor(s) - identity preserved |
| `@Element2` | Picasso image | Picasso + scene visual reference |

---

## Design Rationale

### Consolidated From
- v2.4 Reve Remix prompt (creative vision, interaction style)
- v2.4 Kie.ai video prompt (animation intent)
- User requirements (Spanish accent, era-appropriate banter, ambient sounds)

### Key Decisions
1. **Visual cues from @Element2** - Tone, coloring, texture, lighting derived from Picasso reference image
2. **9:16 vertical format** - Native to Instagram Reels, TikTok, Stories
3. **10 second duration** - Sufficient for meaningful interaction, shareable on all platforms
4. **Natural camera movement** - Intimate, personal feel (push in, drift around)
5. **Era-appropriate dialogue** - "Talented scoundrels" and playful banter of Picasso's time

### What This Prompt Does NOT Do
- Preserve exact background from reference (Kling generates approximate scene)
- Specify hardcoded environment (takes cues from image instead)

---

## Social Media Compatibility

| Platform | 9:16 Support | 10s Duration |
|----------|--------------|--------------|
| TikTok | Native | Native |
| Instagram Reels | Native | Native |
| Instagram Stories | Native | Native (splits if needed) |
| X (Twitter) | Supported | Supported |
| YouTube Shorts | Native | Native |

---

## Related Documents

- `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` - Full node configuration
- `docs/design/docs-design-Consolidated_Prompts_from_v2.4-v1.0-2026_01_08.md` - Source prompts
- `docs/research/docs-research-Kling_O1_Background_Analysis-v1.0-2026_01_08.md` - Background preservation research

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-08 | v1.0 | Initial finalized prompt for v3.0 workflow |
