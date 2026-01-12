# Changelog: Picture with Picasso v2.4 → v3.0

**Version:** v1.0-2026-01-08
**Migration Status:** Ready for Implementation
**Breaking Change:** Yes - Complete API stack replacement

---

## Summary

Major architectural migration replacing the two-stage image composition + video generation pipeline with a single-stage direct video generation approach using Kling O1 Reference-to-Video API.

| Aspect | v2.4 | v3.0 |
|--------|------|------|
| Image Composition | Reve Remix (fal.ai) | Eliminated |
| Video Generation | Kie.ai Wan 2.5 | Kling O1 Reference-to-Video (fal.ai) |
| API Calls | 2 (sequential) | 1 |
| Identity Preservation | Image-based | @Element reference system |
| Aspect Ratio | 16:9 (horizontal) | 9:16 (vertical, social native) |
| Duration | Variable | 10 seconds |

---

## What Changed

### Architecture

**BEFORE (v2.4):**
```
User Image → Resize → Base64 ─┐
                               ├→ Merge → Reve Remix → Poll → Download Image
Picasso Image → Base64 ───────┘
                                              ↓
                               Upload to Cloudinary → Kie.ai Wan 2.5 → Poll → Video
                                              ↓
                               Archive → Route Output → Telegram/Email
```

**AFTER (v3.0):**
```
User Image → Resize → Base64 ─┐
                               ├→ Prepare Kling Elements → Kling O1 → Poll → Video
Picasso Image → Base64 ───────┘
                                              ↓
                               Archive → Route Output → Telegram/Email
```

### Nodes Removed (13 nodes)

| Node Name | Type | Reason |
|-----------|------|--------|
| Merge Images for Reve | Merge | Kling accepts elements directly |
| Send to Reve Remix | HTTP Request | Replaced by Kling API |
| Check Reve Status | HTTP Request | Replaced by Kling status check |
| Check Reve Complete | If | Replaced by Kling completion check |
| Wait for Reve | Wait | Replaced by Kling wait |
| Download Composed Image | HTTP Request | No intermediate image needed |
| Upload Composed Image to Cloudinary | HTTP Request | No Cloudinary needed |
| Create Video Request | HTTP Request | Kie.ai replaced by Kling |
| Get Videos | HTTP Request | Replaced by Kling result fetch |
| If (video status check) | If | Replaced by Kling status router |
| Prepare Video Prompt | Set | Merged into Prepare Kling Elements |
| Wait (video polling) | Wait | Replaced by Kling wait |
| Retry counter nodes | Set/If | Redesigned for Kling flow |

### Nodes Added (10 nodes)

| Node Name | Type | Purpose |
|-----------|------|---------|
| Prepare Kling Elements | Set | Construct API payload with @Element references |
| Submit to Kling | HTTP Request | POST to fal.ai queue endpoint |
| Initialize Retry Counter | Set | Set retryCount = 0 |
| Check Kling Status | HTTP Request | GET status from fal.ai |
| Status Router | Switch | Route by COMPLETED/FAILED/IN_PROGRESS |
| Get Kling Result | HTTP Request | GET completed video metadata |
| Increment Retry Counter | Set | retryCount + 1 |
| Check Max Retries | If | Enforce 30 retry (10 min) timeout |
| Wait for Kling | Wait | 20 second polling interval |
| Download Kling Video | HTTP Request | GET binary video file |

### Prompt Changes

**v2.4 Reve Remix Prompt** (Image Composition - Procedural):
- Detailed instructions for background removal, scaling, positioning
- 5'4" height reference for Picasso
- 16:9 framing requirements
- Focus on image manipulation mechanics

**v3.0 Kling O1 Prompt** (Video Generation - Descriptive):
- Visual style derived from @Element2 (Picasso) reference
- Natural positioning and interaction
- Era-appropriate dialogue ("talented scoundrels")
- Camera movement (push in, drift around)
- 9:16 vertical format for social media
- 10 second duration

### Configuration Changes

| Parameter | v2.4 | v3.0 |
|-----------|------|------|
| `aspect_ratio` | 16:9 | 9:16 |
| `duration` | Not specified | 10 seconds |
| `negative_prompt` | In-prompt text | Dedicated API parameter |
| `polling_interval` | Variable | 20 seconds |
| `max_retries` | Variable | 30 (10 min timeout) |
| Cloudinary | Required | Not required |
| FAL_KEY | Reve Remix | Kling O1 |

### Negative Prompt (v3.0 Addition)

```
face morphing, identity swap, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur
```

Quality controls derived from v2.4 Reve Remix prompt ("Avoid extra or duplicated limbs, distorted anatomy, or motion blur") plus Kling O1 identity preservation best practices.

---

## Rationale

### Why Kling O1?

1. **Single-stage architecture** - Eliminates intermediate image composition step
2. **Native multi-identity support** - @Element system preserves both user and Picasso identities
3. **Cost reduction** - One API call instead of two
4. **Latency improvement** - Faster end-to-end processing
5. **Unified platform** - Both capabilities on fal.ai

### Why 9:16 Vertical Format?

| Platform | 9:16 Support | 10s Duration |
|----------|--------------|--------------|
| TikTok | Native | Native |
| Instagram Reels | Native | Native |
| Instagram Stories | Native | Native (splits if needed) |
| X (Twitter) | Supported | Supported |
| YouTube Shorts | Native | Native |

### Accepted Tradeoffs

| Capability | v2.4 | v3.0 | Accepted? |
|------------|------|------|-----------|
| Exact background preservation | Yes (Reve Remix) | No (AI-generated) | ✅ Yes |
| Historical location accuracy | High | Approximate | ✅ Yes |
| Identity preservation | High | High | ✅ Maintained |
| Social media native format | No (16:9) | Yes (9:16) | ✅ Improvement |

---

## JSON v3.0 Validation Analysis

### Validation Tools Used

The v3.0 node configurations were validated using the following MCP tools:

| Tool | Server | Purpose |
|------|--------|---------|
| `mcp__n8n-mcp__validate_node` | n8n-mcp | Validate individual node configurations |
| `mcp__n8n-mcp__get_node` | n8n-mcp | Get node documentation and schema |
| `mcp__n8n-workflows_Docs__search_n8n_workflows_docs` | n8n-workflows Docs | Search community workflow patterns |
| `mcp__n8n-mcp__search_templates` | n8n-mcp | Search official n8n templates |

### Node Validation Results

| Node Type | Validation Status | Errors | Warnings | Suggestions |
|-----------|-------------------|--------|----------|-------------|
| `n8n-nodes-base.httpRequest` | ✅ Valid | 0 | 0 | 1 (alwaysOutputData for error handling) |
| `n8n-nodes-base.set` | ✅ Valid | 0 | 1* | 0 |
| `n8n-nodes-base.switch` | ✅ Valid | 0 | 0 | 0 |
| `n8n-nodes-base.wait` | ✅ Valid | 0 | 0 | 0 |
| `n8n-nodes-base.if` | ✅ Valid | 0 | 0 | 0 |

*Set node warning is for base config validation only; actual nodes include field assignments.

### Validation Details

#### HTTP Request Nodes (4 nodes)
- **Submit to Kling**: POST to `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video`
- **Check Kling Status**: GET status endpoint with request_id
- **Get Kling Result**: GET completed result with video URL
- **Download Kling Video**: GET binary file from video.url

**Suggestion Applied**: Consider adding `alwaysOutputData: true` at node level for better error handling.

#### Set Nodes (3 nodes)
- **Prepare Kling Elements**: Constructs `elements` array with @Element references, prompt, duration, aspect_ratio
- **Initialize Retry Counter**: Sets `retryCount = 0` and preserves `request_id`
- **Increment Retry Counter**: Increments `retryCount` by 1

#### Switch Node (1 node)
- **Status Router**: Routes by status value (COMPLETED, FAILED, IN_QUEUE, IN_PROGRESS)
- **Fallback**: Extra output for unexpected status values

#### Wait Node (1 node)
- **Wait for Kling**: 20-second interval (under 65s threshold per n8n docs - no DB offload)

#### If Node (1 node)
- **Check Max Retries**: `retryCount >= 30` (10 minute timeout)

### Documentation References

Node configurations were designed based on official n8n documentation:

| Node | Documentation Source |
|------|---------------------|
| HTTP Request | n8n-mcp `get_node` mode=docs - Method, URL, Headers, Body, Response options |
| Switch | n8n-mcp `get_node` mode=docs - Rules mode with named outputs |
| Wait | n8n-mcp `get_node` mode=docs - Time interval under 65s for no DB offload |
| Set | n8n-mcp `get_node` mode=docs - Manual mode with assignments |
| If | n8n-mcp `get_node` mode=docs - Conditions with number comparison |

### Type Versions Used

| Node Type | typeVersion | Notes |
|-----------|-------------|-------|
| HTTP Request | 4.2 | Current stable version |
| Set | 3.4 | Current stable version |
| Switch | 3.2 | Current stable version |
| Wait | 1.1 | Current stable version |
| If | 2.2 | Current stable version |

### Nodes to Remove (Complete List with IDs)

The following 13 nodes must be removed from v2.4 before inserting v3.0 nodes:

| # | Node Name | Node ID | Type |
|---|-----------|---------|------|
| 1 | Merge Images for Reve | `bf48af0c-1dff-447d-ae18-a0e1e760183b` | Merge |
| 2 | Send to Reve Remix | `a2ca9aae-ac85-4233-8f82-0d9132c29117` | HTTP Request |
| 3 | Check Reve Status | `ad4c3c69-807f-485c-b102-6365c5cbf368` | HTTP Request |
| 4 | Check Reve Complete | `f3533944-d0d8-4e8f-80e9-5bea075b6477` | If |
| 5 | Wait for Reve | `00a70002-9632-4244-8183-747222a96dd5` | Wait |
| 6 | Download Composed Image | `d7f1063b-bc6a-43c1-b7ba-fb649ea213ee` | HTTP Request |
| 7 | Upload Composed Image to Cloudinary | `7d35077f-b2e7-483c-aa45-4a27579f5c25` | HTTP Request |
| 8 | Archive Composed Image to Drive | `02513ee7-5d42-459a-8401-59aa0192a1fd` | Google Drive |
| 9 | Prepare Video Prompt | `b4e9f649-28e3-4a8e-880b-51af617fef46` | Set |
| 10 | Create Video Request | `792af95e-9fe1-4fc5-b178-cc90b9aa08dc` | HTTP Request |
| 11 | Get Videos | `6c50a923-ea4e-4c9a-a5fd-f13b86032916` | HTTP Request |
| 12 | If (video status) | `93404b14-c0cf-4977-8fd5-eb5dcefa39ff` | If |
| 13 | Wait (video polling) | `14946e06-976c-44e0-8a8e-82365bb38e08` | Wait |

### Connection Points

#### Insertion Points (connect FROM these existing nodes)

| Existing Node | Node ID | Connect TO |
|---------------|---------|------------|
| Convert User Image to Base64 | `1f11a417-39c6-40ef-b68e-c868cdd0f226` | Prepare Kling Elements (input 0) |
| Convert Picasso Image to Base64 | `87446f7f-cc49-45ce-8261-14ffb5875ea9` | Prepare Kling Elements (input 1) |

#### Exit Points (connect TO these existing nodes)

| New Node | Connect TO | Node ID |
|----------|------------|---------|
| Download Kling Video | Archive Video to Google Drive | `c0b9321d-be63-4e41-8cd4-5bbf190839cf` |
| Download Kling Video | Route Output | `a862b8a9-22f1-494a-9738-d25eda420867` |
| Check Max Retries (TRUE) | Error handling (user to add) | N/A |

---

## Supporting Documents

### Analysis Documents

| Document | Purpose |
|----------|---------|
| `docs/analysis/analysis-workflow-reconfiguration-plan-v1.0-2026-01-06.md` | Complete implementation plan with node configurations |
| `docs/analysis/analysis-tool-comparative-analysis-v1.0-2026-01-06.md` | Tool comparison research (Kling vs alternatives) |

### Design Documents

| Document | Purpose |
|----------|---------|
| `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` | Kling O1 node configurations |
| `docs/design/docs-design-Consolidated_Prompts_from_v2.4-v1.0-2026_01_08.md` | Extracted prompts from v2.4 for reference |
| `docs/design/design-wan-2.6-configuration-v1.0-2026-01-06.md` | Alternative WAN 2.6 config (not selected) |

### Research Documents

| Document | Purpose |
|----------|---------|
| `docs/research/docs-research-Kling_O1_Background_Analysis-v1.0-2026_01_08.md` | Background preservation capability research |

### Generated Output Files

| File | Purpose |
|------|---------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.0-2026_01_08.md` | Finalized Kling O1 prompt |
| `source/workflows/source-workflows-Kling_O1_Nodes_Insert-v1.0-2026_01_08.json` | Standalone JSON for manual node insertion |
| `source/components/source-components-Kling_O1_v3.0_Configuration-v1.0-2026_01_08.md` | Complete v3.0 configuration reference |
| `docs/guides/docs-guides-Kling_O1_v3.0_Implementation_Guide-v1.0-2026_01_08.md` | Step-by-step implementation guide |

### External References

| Source | URL |
|--------|-----|
| Kling O1 Reference-to-Video API | https://fal.ai/models/fal-ai/kling-video/o1/reference-to-video/api |
| Kling O1 Prompt Guide | https://fal.ai/learn/devs/kling-o1-prompt-guide |
| Kling O1 Developer Guide | https://fal.ai/learn/devs/kling-o1-developer-guide |

---

## Implementation Instructions

### Manual Node Insertion Process

1. **Backup current workflow**
   - Export `Picture with Picasso -Wan 2.5-v2.4.json` from n8n
   - Save to `versions/workflows/`

2. **Import new nodes**
   - Open `source/workflows/source-workflows-Kling_O1_Nodes_Insert-v1.0-2026_01_08.json`
   - Copy nodes array into workflow

3. **Remove old nodes** (13 nodes listed in `_nodesToRemove` section)

4. **Connect insertion points**
   - `Convert User Image to Base64` → `Prepare Kling Elements` (input 0)
   - `Convert Picasso Image to Base64` → `Prepare Kling Elements` (input 1)

5. **Connect exit points**
   - `Download Kling Video` → `Archive Video to Google Drive`
   - `Download Kling Video` → `Route Output`

6. **Update workflow metadata**
   - Rename to `Picture with Picasso v3.0`
   - Update version in workflow settings

7. **Configure credentials**
   - Ensure FAL_KEY environment variable is set
   - Verify fal.ai account has Kling O1 access

8. **Test end-to-end**
   - Run with test images
   - Verify video output and archival

---

## Rollback Plan

If issues arise with v3.0:

1. Restore `Picture with Picasso -Wan 2.5-v2.4.json` from `versions/workflows/`
2. Import into n8n
3. Activate restored workflow
4. Deactivate v3.0 workflow

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-11 | v1.9.2 | Group reunion dynamics, ALL-to-ALL interaction, strengthened identity preservation |
| 2026-01-09 | v1.5 | Added v3.0.5 - Fixed 2500 char limit (was 5600→2383), added period audio prohibitions |
| 2026-01-09 | v1.4 | Added v3.0.4 - Historical fidelity, identity preservation, visual master constraints |
| 2026-01-09 | v1.3 | Added v3.0.3 audio direction (vocalizations, expressions, lip sync - optimized for Kling-Foley) |
| 2026-01-09 | v1.2 | Added v3.0.2 prompt update (old friends reunion narrative) |
| 2026-01-09 | v1.1 | Added v3.0.1 implementation fixes (generate_audio, polling, routing) |
| 2026-01-08 | v1.0 | Initial changelog documenting v2.4 → v3.0 migration |

---

## v3.0.5 Character Limit Fix & Period Audio (2026-01-09)

### Root Cause: Execution 3981 Failure

**Error:** `422 - ensure this value has at most 2500 characters`

The v1.4 prompt exceeded fal.ai's 2500 character limit for the main prompt field.

| Version | Main Prompt | Limit | Status |
|---------|-------------|-------|--------|
| v1.4 | ~5,600 chars | 2,500 | FAILED |
| v1.5 | ~2,383 chars | 2,500 | OK |

### Reduction Strategy

**Removed (moved to negative prompt):**
- DO NOT section (already covered by negative prompt)
- Verbose if/then structures
- Separate PICASSO/VISITORS subsections

**Preserved (condensed):**
- Core reunion narrative
- @Element2 as visual master
- Identity preservation requirements
- Audio generation directives
- Technical specifications

**Added back (within limit):**
- Enhanced INTERACTION section (banter, teasing, emotional warmth)
- Detailed ambient sounds (studio, café, outdoor specifics)
- CAMERA section (gentle push, slow drift)

### New Negative Prompt Additions

Added period-appropriate audio prohibitions:
```
electronic sounds, digital audio, contemporary music, modern background noise
```

### Files Updated

| File | Change |
|------|--------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.5-2026_01_09.md` | New condensed prompt |

### Manual Workflow Updates Required

| Node | Field | Action |
|------|-------|--------|
| Prepare Kling Elements1 | prompt | Replace with v1.5 prompt (~2,383 chars) |
| Prepare Kling Elements1 | negative_prompt | Replace with v1.5 negative prompt |
| Submit to Kling1 | jsonBody | Ensure `"generate_audio": true` is present |

---

## v3.0.4 Historical Fidelity & Identity Preservation (2026-01-09)

### Issues Identified in v1.3 Testing

| Issue | Root Cause |
|-------|------------|
| Picasso had mustache (he was clean-shaven) | No explicit facial feature preservation directive |
| Modern color grading | No era-specific color/texture vocabulary |
| Modern fixtures in background | No environment fidelity constraints |
| Man bun invented for visitor | No "do not invent unseen features" constraint |
| No audio | Missing `generate_audio: true` in API request |

### Solution: Research-Backed Prompt Restructure

Based on Perplexity research on Kling O1 best practices:

1. **Semantic Anchoring** - Dense historical visual descriptors
2. **Constraint Sandwich Architecture** - Layered prompt structure
3. **Explicit Preservation Directives** - What MUST be kept from source
4. **Period-Specific Negative Prompts** - Not generic exclusions

### Key Principle: @Element2 as Visual Master

The Picasso reference image (@Element2) defines ALL visual characteristics:

| If @Element2 is... | Video is... |
|-------------------|-------------|
| Color | Color (same palette) |
| Sepia | Sepia (same tonal range) |
| Black & White | Black & White |
| Grainy | Grainy (same texture) |

Visitors from @Element1 are placed INTO @Element2's visual world with its color/tone/grain applied, but their actual physical appearance remains unchanged.

### New Prompt Sections

#### VISUAL MASTER Section
```
COLOR & TONE:
- If @Element2 is COLOR → video is color, matching that exact color palette
- If @Element2 is SEPIA → video is sepia with that same tonal range
- If @Element2 is BLACK & WHITE → video is black and white

TEXTURE & GRAIN:
- If @Element2 has FILM GRAIN → video retains that grain texture
- PRESERVE the photographic texture - do NOT smooth to modern clarity

LIGHTING:
- Match the EXACT lighting direction from @Element2
- Match the EXACT shadow depth and contrast

ENVIRONMENT:
- Preserve the architecture, furnishings, surfaces exactly as shown
- DO NOT add any elements not present in @Element2
```

#### ABSOLUTE IDENTITY PRESERVATION Section
```
ALL PEOPLE MUST REMAIN EXACTLY AS THEY APPEAR IN THEIR SOURCE IMAGES:

PICASSO (@Element2):
- Face must match @Element2 EXACTLY - no modifications whatsoever
- If clean-shaven → remains clean-shaven (NO mustache, NO beard, NO goatee)
- Same hairstyle, hairline, facial structure, clothing
- Do NOT age or de-age

VISITORS (@Element1):
- Faces must match @Element1 EXACTLY - no modifications whatsoever
- Same hairstyle, hair color, hair length, clothing
- Do NOT invent features for parts not visible
- Do NOT age or de-age
```

#### DO NOT Section (Explicit Prohibitions)
```
IDENTITY:
- DO NOT add facial hair to clean-shaven faces
- DO NOT change anyone's hairstyle from their source image
- DO NOT change anyone's clothing
- DO NOT age or de-age anyone
- DO NOT invent features for unseen parts

VISUAL:
- DO NOT apply modern color grading or filters
- DO NOT convert color/tone to something different from @Element2
- DO NOT smooth out film grain or vintage texture
- DO NOT change the lighting direction from @Element2
```

### Enhanced Negative Prompt

Added identity-specific negatives:
```
identity change, added facial hair, removed facial hair, mustache on clean-shaven face,
beard on clean-shaven face, goatee on clean-shaven face, changed hairstyle, different hairstyle,
changed hair color, different hair length, changed clothing, different clothing, aged face,
de-aged face, added wrinkles, removed wrinkles, invented features, modern color grading,
Instagram filter, converted color tone, digital smoothing, smoothed grain, removed grain,
modern clarity, added environment elements, changed lighting direction
```

### Audio Fix (Critical)

**Root Cause:** `generate_audio: true` was missing from API request body.

**Fix Location:** `Submit to Kling1` node → jsonBody field

**Before:**
```javascript
={{ JSON.stringify({
  "prompt": $json.prompt,
  "negative_prompt": $json.negative_prompt,
  "elements": $json.elements,
  "duration": $json.duration,
  "aspect_ratio": $json.aspect_ratio
}) }}
```

**After:**
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

### Files Updated

| File | Change |
|------|--------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.4-2026_01_09.md` | New prompt version with historical fidelity |

### Manual Workflow Updates Required

| Node | Field | Action |
|------|-------|--------|
| Prepare Kling Elements1 | prompt | Replace with v1.4 prompt |
| Prepare Kling Elements1 | negative_prompt | Replace with v1.4 negative prompt |
| Submit to Kling1 | jsonBody | Add `"generate_audio": true` |

### Testing Checklist

- [ ] Audio is present (vocalizations, ambient sounds)
- [ ] Picasso matches source exactly (no added mustache if clean-shaven)
- [ ] Color tone matches @Element2 exactly (color/sepia/B&W preserved)
- [ ] Grain and texture matches @Element2 (not smoothed)
- [ ] Environment maintains period authenticity (no modern fixtures)
- [ ] Lighting direction matches @Element2
- [ ] Visitor features match source (no invented hairstyles, no changed clothing)

---

## v3.0.3 Audio & Vocalization Direction (2026-01-09)

### Audio Strategy: Optimized for Kling-Foley

**Research Finding (Perplexity):** Kling-Foley is designed for music/FX/ambience, not reliable intelligible dialogue. Full speech with accents is unreliable.

**Approach:** Focus on expressive vocalizations that Kling-Foley handles well:
- ✅ Laughter, exclamations, emotional sounds
- ✅ Short impulsive expressions ("Hey!", "Yes!", cheers)
- ✅ Ambient sounds (café, studio, outdoor)
- ✅ Happy sounds during embraces
- ❌ Avoided: Full dialogue, complex sentences, precise accent control

### Changes Made

#### New AUDIO Section (Vocalization-Focused)

```
AUDIO:
- Warm, expressive vocalizations - laughter, exclamations, emotional sounds
- Short impulsive expressions: "Hey!", "Yes!", "Ah, my friend!", cheers, hoots
- Picasso's voice has Spanish-accented warmth and energy
- Happy sounds during embraces - sighs of joy, delighted gasps
- Playful teasing sounds - affectionate "talented scoundrel!" type exclamations
- Ambient sounds authentic to the scene (café chatter, studio sounds, outdoor atmosphere)
- Lip movements match vocalizations naturally
```

#### INTERACTION Section Simplified

| v1.2 | v1.3 |
|------|------|
| Scripted dialogue examples | Spontaneous expressions of joy |
| "References to shared memories" | "The energy of friends who haven't seen each other in a long time" |

#### TECHNICAL Section Updated

Added: "Realistic talking mouth movements with clear lip sync"

#### Negative Prompt Additions

```
silent video, no audio, out of sync lip movements, mismatched audio, closed mouths while speaking
```

### Files Updated

| File | Change |
|------|--------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.3-2026_01_09.md` | New prompt version |
| Prepare Kling Elements1 | Update prompt field with v1.3 content |

### Rationale

This approach:
1. Works within Kling-Foley's actual capabilities
2. Captures emotional energy of reunion without depending on precise speech
3. Maintains Picasso's Spanish-accented warmth in tone/energy
4. Can be enhanced with ElevenLabs TTS later if full dialogue needed

---

## v3.0.2 Prompt Update (2026-01-09)

### Narrative Reframe: Old Friends Reunion

**Issue:** v1.0/v1.1 prompts treated Picasso primarily as a style reference. The original v2.4 intent was for Picasso to be an actively animated character "horsing around" with visitors.

**Creative Direction:** Picasso and visitors are old friends who haven't seen each other in a long time - happy, playful, and chummy when they reunite.

### Prompt Changes

| Section | v1.1 | v1.2 |
|---------|------|------|
| Opening | "close friends joyfully chumming around" | "old friends reuniting after a long time apart... overjoyed to see each other" |
| New Section | N/A | "CHARACTERS" section emphasizing equal animation |
| Picasso Role | Implied style reference | "fully animated and expressive - not a background figure" |
| Interaction | "animated conversation, shared laughter" | "embracing, laughing, horsing around... excited greetings, happy embraces, playful banter" |
| Picasso Actions | Not specified | "laughing, gesturing expressively, clapping friends on the back" |

### Negative Prompt Additions

Added to prevent static/frozen Picasso:
```
static Picasso, frozen characters, stiff poses
```

### Files Updated

| File | Change |
|------|--------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.2-2026_01_09.md` | New prompt version |
| Prepare Kling Elements1 | Update prompt field with v1.2 content |

### Original v2.4 Reference

From `docs/design/docs-design-Consolidated_Prompts_from_v2.4-v1.0-2026_01_08.md`:
> "Place them into Image 2 so they appear to be **very close friends** with the existing **Picasso character or characters**: standing or sitting close together, **facing toward each other** with **warm, playful, emotionally connected body language**, as if they are **joyfully chumming around and having fun**."

---

## v3.0.1 Implementation Fixes (2026-01-09)

### Audio Generation

**Issue:** Generated videos had no audio (speech or ambient sounds).

**Fix:** Added `generate_audio: true` parameter.

| Node | Change |
|------|--------|
| Prepare Kling Elements1 | Added `generate_audio` = `true` (Boolean) |
| Submit to Kling1 | Updated jsonBody to include `"generate_audio": $json.generate_audio` |

**Prompt Updated:** `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.1-2026_01_09.md`

---

### Polling Loop Fixes

**Issue 1:** Check Kling Status1 using wrong URL format.

| Field | Before | After |
|-------|--------|-------|
| URL | Hardcoded with `/o1/reference-to-video/` path | `={{ $json.status_url }}` |

**Issue 2:** Get Kling Result1 using wrong URL format.

| Field | Before | After |
|-------|--------|-------|
| URL | Hardcoded with `/o1/reference-to-video/` path | `={{ $json.response_url }}` |

**Issue 3:** Dual authentication headers causing 405 errors.

| Node | Change |
|------|--------|
| Check Kling Status1 | Changed Authentication from `Predefined Credential Type` to `None` |
| Get Kling Result1 | Changed Authentication from `Predefined Credential Type` to `None` |

**Issue 4:** retryCount lost during polling loop.

| Solution | Description |
|----------|-------------|
| New Node | Added "Preserve Retry Count" Set node after Check Kling Status1 |
| Expression | `={{ $('Increment Retry Counter1').first().json.retryCount ?? $('Initialize Retry Counter1').first().json.retryCount ?? 0 }}` |
| Connection | Check Kling Status1 → Preserve Retry Count → Status Router1 |

---

### Output Routing Fixes

**Issue:** inputSource and chatId fields lost after Kling processing, causing wrong routing.

| Node | Field | Before | After |
|------|-------|--------|-------|
| Route Output | Condition Left Value | `{{ $json.inputSource }}` | `{{ $('Workflow Configuration').first().json.inputSource }}` |
| Send to Telegram | Chat ID | `{{ $json.chatId }}` | `{{ $('Workflow Configuration').first().json.chatId }}` |

---

### Element Mapping Reference

| Element | Source | API Reference |
|---------|--------|---------------|
| `@Element1` | User photo (base64) | `frontal_image_url`, `reference_image_urls` |
| `@Element2` | Picasso image (base64) | `frontal_image_url`, `reference_image_urls` |

---

### Complete Parameter Set (v3.0.1)

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `prompt` | See prompt file | Video generation instructions |
| `negative_prompt` | 8 items | Quality control |
| `elements` | Array of 2 | @Element1 (user), @Element2 (Picasso) |
| `duration` | `10` | 10 second video |
| `aspect_ratio` | `9:16` | Vertical, social media native |
| `generate_audio` | `true` | Native Kling-Foley audio |

---

## v1.9.2 Group Reunion Dynamics & Identity Strengthening (2026-01-11)

### Issues Identified in v1.9.1 Testing

| Issue | Root Cause |
|-------|------------|
| Generic hand gestures only | Interaction buried mid-prompt, model ignores it |
| People standing apart | No explicit "physical contact required" directive |
| Picasso-to-visitor only | No ALL-to-ALL interaction language |
| Visitor likeness drift | Identity section too low in prompt |
| Formal/stiff energy | "platonic" and passive language |

### Solution: Major Prompt Restructure

**Key Principle: Primacy Effect**

I2V models heavily weight the BEGINNING of prompts. Moved interaction to FIRST section.

### Prompt Changes

#### Structure Reorder

| Section | v1.9.1 Position | v1.9.2 Position |
|---------|-----------------|-----------------|
| Interaction | Paragraph 7 | **FIRST** |
| Identity | Paragraph 2 | Second |
| Visual Fidelity | Paragraph 1 | Third |
| Motion | Paragraph 5 | Fourth |

#### New GROUP REUNION Section (First in Prompt)

```
GROUP REUNION – THE HEART OF THIS VIDEO:
Everyone in this frame—@Element1 (visitors) and @Element2 (Picasso)—are great
friends who haven't seen each other in years. This is the surprise reunion.
They are ecstatic. They can't believe it.

ALL figures interact with EACH OTHER, not just with Picasso:
- Visitors embrace each other AND Picasso
- Group hugs, arms around multiple shoulders
- Friends grabbing friends, pulling them closer
- Shared laughter rippling through the group
- Playful shoving, back-patting, hand-clasping between everyone
- The energy of "I can't believe you're HERE!"

Physical contact between ALL parties is REQUIRED. No one stands apart.
No polite distance. No observer figures. Everyone is IN the reunion.
```

#### Identity Section Strengthened

Added:
- "NON-NEGOTIABLE" emphasis
- "Lock: face shape, facial features, skin tone, hair, clothing, accessories"
- "No drift, no morphing, no interpretation"

#### Removed

| Term | Reason |
|------|--------|
| "platonic" | User request - clinical/cold connotation |
| "MOTION ORIGIN" section | Was anchoring model to static source pose |
| "BEHAVIORAL VARIABILITY" section | Replaced with group dynamics |

### Negative Prompt Additions

Added 15 new anti-isolation terms:

```
standing apart, isolated figures, polite distance, observer posture,
watching from sidelines, one-on-one only, ignoring other visitors,
separate conversations, formal arrangement, posed group photo, stiff lineup,
hands raised without purpose, distant body language, formal posture, static figures
```

### Files Created

| File | Purpose |
|------|---------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.9.2-2026_01_11.md` | New video prompt |
| `source/prompts/source-prompts-Kling_O1_Negative_Prompt-v1.9.2-2026_01_11.md` | New negative prompt |

### Manual Workflow Updates Required

| Node | Field | Action |
|------|-------|--------|
| Prepare Kling Elements1 | prompt | Replace with v1.9.2 prompt |
| Prepare Kling Elements1 | negative_prompt | Replace with v1.9.2 negative prompt |

### Optional Enhancement: Interaction Randomization

Add a Code node before prompt assembly for per-video interaction variety:

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

### Testing Checklist

- [ ] All figures interact with each other (not just Picasso)
- [ ] Physical contact present between multiple parties
- [ ] No isolated/observer figures standing apart
- [ ] Visitor faces match source exactly (no drift)
- [ ] Picasso matches source exactly
- [ ] Energy is ecstatic reunion, not formal greeting
- [ ] Group hugs, embraces, playful gestures present

### Future Enhancement: User-Directed Interaction

Phase 2 option: Allow users to submit interaction preferences with their photo:

| Input | Effect |
|-------|--------|
| User provides "dancing together" | Injected into video prompt |
| User provides nothing | Random selection from curated list |

---

## Related Session Files

- `AA-01-WHERE-WE-ARE-NOW.md` - Current project state
- `AA-04-DECISION-LOG.md` - Decision history including Kling O1 selection
