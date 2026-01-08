# Picture with Picasso - Workflow Reconfiguration Plan

**Version:** v1.0-2026-01-06
**Status:** Ready for Implementation
**Workflow Target Version:** v3.0

---

## Overview

**Objective:** Replace Reve Remix + Kie.ai Wan 2.5 with Kling O1 Reference-to-Video (fal.ai) for direct multi-identity video generation.

**Key Benefit:** Skip intermediate image composition step - generate video directly from reference images while preserving both user and Picasso identities.

**Workflow File:** `source/workflows/Picture with Picasso -Wan 2.5-v2.4.json`

---

## User Decisions (Confirmed)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Archive Strategy | Keep as parallel process | Archive reference images while video generates |
| Polling Interval | **20 seconds** | Under 65s threshold - no database offload (n8n docs) |
| Error Handling | Retry + Log + Notify | 3 retries, log to Google Sheets, notify user on final failure |
| Version | **v3.0** | Major architectural change - new API stack |

---

## Research Sources

| Source | URL | Purpose |
|--------|-----|---------|
| fal.ai API Docs | https://fal.ai/models/fal-ai/kling-video/o1/reference-to-video/api | Kling Reference-to-Video schema |
| n8n Wait Node Docs | n8n-mcp (local) | Wait times <65s don't offload to database |
| n8n-mcp | Local MCP server | HTTP Request node patterns |
| Existing Config | `docs/guides/Config-*.md` | Current workflow patterns |

---

## Current Architecture (BEFORE)

```
User Image → Resize → Base64 ─┐
                               ├→ Merge → Reve Remix → Poll → Download Image
Picasso Image → Base64 ───────┘
                                              ↓
                               Upload to Cloudinary → Kie.ai Wan 2.5 → Poll → Video
                                              ↓
                               Archive → Route Output → Telegram/Email
```

**Current Node Count:** 41 nodes, 29 connections

---

## Target Architecture (AFTER)

```
User Image → Resize → Base64 ─┐
                               ├→ Prepare Kling Elements → Kling Reference-to-Video → Poll → Video
Picasso Image → Base64 ───────┘
                                              ↓
                               Archive → Route Output → Telegram/Email
```

**Expected Node Count:** ~40 nodes (removes 10, adds 10)

---

## Phase 1: Nodes to REMOVE (10 nodes)

### Reve Remix Pipeline (6 nodes)
| Node Name | Type | Reason |
|-----------|------|--------|
| Merge Images for Reve | n8n-nodes-base.merge | No longer needed - Kling accepts elements directly |
| Send to Reve Remix | n8n-nodes-base.httpRequest | Replaced by Kling API |
| Check Reve Status | n8n-nodes-base.httpRequest | Replaced by Kling status check |
| Check Reve Complete | n8n-nodes-base.if | Replaced by Kling completion check |
| Wait for Reve | n8n-nodes-base.wait | Replaced by Kling wait |
| Download Composed Image | n8n-nodes-base.httpRequest | No intermediate image needed |

### Kie.ai Video Pipeline (4 nodes)
| Node Name | Type | Reason |
|-----------|------|--------|
| Upload Composed Image to Cloudinary | n8n-nodes-base.httpRequest | No Cloudinary needed |
| Create Video Request | n8n-nodes-base.httpRequest | Kie.ai replaced by fal.ai Kling |
| Get Videos | n8n-nodes-base.httpRequest | Replaced by Kling result fetch |
| If (video status check) | n8n-nodes-base.if | Replaced by Kling completion check |

---

## Phase 2: Nodes to ADD (10 nodes)

### 1. Prepare Kling Elements
**Type:** n8n-nodes-base.set
**Position:** After Base64 conversion nodes
**Configuration:**
```json
{
  "elements": [
    {
      "reference_image_urls": ["{{ $('Convert User Image to Base64').item.json.userImageBase64 }}"],
      "frontal_image_url": "{{ $('Convert User Image to Base64').item.json.userImageBase64 }}"
    },
    {
      "reference_image_urls": ["{{ $('Convert Picasso Image to Base64').item.json.picassoImageBase64 }}"],
      "frontal_image_url": "{{ $('Convert Picasso Image to Base64').item.json.picassoImageBase64 }}"
    }
  ],
  "prompt": "[Combined prompt - see Prompt Section below]",
  "duration": "5",
  "aspect_ratio": "16:9"
}
```

### 2. Submit to Kling Reference-to-Video
**Type:** n8n-nodes-base.httpRequest
**Method:** POST
**URL:** `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video`
**Headers:**
```
Authorization: Key {{ $env.FAL_KEY }}
Content-Type: application/json
```
**Body:**
```json
{
  "prompt": "={{ $json.prompt }}",
  "elements": "={{ $json.elements }}",
  "duration": "={{ $json.duration }}",
  "aspect_ratio": "={{ $json.aspect_ratio }}"
}
```
**Output:** `request_id`

### 3. Check Kling Status
**Type:** n8n-nodes-base.httpRequest
**Method:** GET
**URL:** `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/{{ $json.request_id }}/status`
**Headers:**
```
Authorization: Key {{ $env.FAL_KEY }}
```
**Output:** `status` ("IN_QUEUE", "IN_PROGRESS", "COMPLETED", "FAILED")

### 4. Kling Complete?
**Type:** n8n-nodes-base.if
**Condition:** `{{ $json.status }}` equals `COMPLETED`
- **TRUE:** Connect to Get Kling Result
- **FALSE:** Connect to Wait for Kling

### 5. Wait for Kling
**Type:** n8n-nodes-base.wait
**Duration:** 20 seconds (under 65s threshold - no DB offload per n8n docs)
**Resume:** Loop back to Check Kling Status

### 6. Increment Retry Counter
**Type:** n8n-nodes-base.set
**Configuration:**
```json
{
  "retryCount": "={{ ($json.retryCount || 0) + 1 }}"
}
```
**Purpose:** Track polling attempts for timeout protection

### 7. Check Max Retries
**Type:** n8n-nodes-base.if
**Condition:** `{{ $json.retryCount }}` >= 30 (30 × 20s = 10 min max)
- **TRUE:** Route to Log Error + Notify User
- **FALSE:** Continue to Check Kling Status

### 8. Get Kling Result
**Type:** n8n-nodes-base.httpRequest
**Method:** GET
**URL:** `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/{{ $json.request_id }}`
**Output:** `video.url`

### 9. Log Error to Google Sheets
**Type:** n8n-nodes-base.googleSheets
**Operation:** Append Row
**Spreadsheet:** Workflow Error Log (create or specify ID)
**Sheet:** Kling Errors
**Columns:**
```json
{
  "timestamp": "={{ $now }}",
  "workflow": "Picture with Picasso v3.0",
  "error_type": "={{ $json.status === 'FAILED' ? 'API_FAILURE' : 'TIMEOUT' }}",
  "request_id": "={{ $json.request_id }}",
  "retry_count": "={{ $json.retryCount }}",
  "user_source": "={{ $json.inputSource }}",
  "user_id": "={{ $json.inputSource === 'telegram' ? $json.chatId : $json.userEmail }}"
}
```
**Trigger:** On FAILED status OR max retries exceeded

### 10. Notify User of Failure
**Type:** n8n-nodes-base.if (route by input source)
- **Telegram:** Send error message via Telegram node
- **Form:** Send error email via Gmail node
**Message Template:**
```
Sorry, we encountered an issue generating your video with Picasso.
Our team has been notified. Please try again later.
Error Reference: {{ $json.request_id }}
```

---

## Phase 3: Nodes to MODIFY (2 nodes)

### 1. Prepare Video Prompt → DELETE
- Replaced by "Prepare Kling Elements" node

### 2. Archive Composed Image to Drive → RENAME
- New name: "Archive Reference Images" (optional)
- Or DELETE if archival not needed

---

## Phase 4: Connection Rewiring

### New Connection Flow
```
Convert User Image to Base64 ──────┐
                                    ├→ Prepare Kling Elements
Convert Picasso Image to Base64 ───┘
                                    │
                    ┌───────────────┴───────────────┐
                    ↓ (PARALLEL)                    ↓
        Archive Reference Images          Submit to Kling Reference-to-Video
        to Google Drive                             ↓
                                          Check Kling Status ←─────────────┐
                                                    ↓                      │
                                          Kling Complete?                  │
                                           /    |    \                     │
                                    COMPLETED FAILED  IN_PROGRESS          │
                                         ↓      ↓         ↓                │
                                    Get Kling  Log     Wait for Kling      │
                                    Result   Error     (20 seconds)        │
                                         ↓      ↓         ↓                │
                                         │      │    Increment Retry       │
                                         │      │    Counter               │
                                         │      │         ↓                │
                                         │      │    Check Max Retries?    │
                                         │      │     /        \           │
                                         │      │   YES        NO ─────────┘
                                         │      ↓    ↓
                                         │    Log Error to Google Sheets
                                         │           ↓
                                         │    Notify User of Failure
                                         │           ↓
                                         │    Route Error Output
                                         │     /           \
                                         │  Telegram      Email
                                         ↓
                              Archive Video to Google Drive
                                         ↓
                                    Route Output
                                     /        \
                              Telegram      Email
```

### Parallel Archive Flow
The archive of reference images runs **in parallel** with the Kling submission:
- Does NOT block video generation
- Uses existing Google Drive archive node (renamed)

---

## Combined Prompt Template

```
Create a video where @Element1 (the user) and @Element2 (Picasso) are having a warm, friendly interaction in a Picasso-style artistic environment.

@Element1 represents a real person who should maintain their exact facial identity and recognizable features throughout the video.

@Element2 represents Picasso who should maintain his iconic appearance and artistic persona.

Scene requirements:
- Setting: A Picasso-style artistic environment with bold colors, geometric shapes, and cubist influences matching Picasso's era and artistic style
- Interaction: The two subjects should appear as close friends - standing or sitting together, facing each other with warm, playful, emotionally connected body language
- Animation: Show them joyfully interacting, perhaps gesturing, laughing, or having a conversation as if they are old friends
- Style: Maintain consistent Picasso-inspired artistic atmosphere throughout

Technical requirements:
- Keep both subjects' identities stable and recognizable
- Natural proportions and scale
- Smooth, cinematic motion
```

---

## Configuration Updates

### Environment Variables
| Variable | Current | Change |
|----------|---------|--------|
| FAL_KEY | Used for Reve | Keep - also works for Kling |
| KIE_AI_KEY | Used for Wan 2.5 | Can be removed (optional) |

### Workflow Configuration Node Updates
```json
{
  "duration": "5",
  "aspect_ratio": "16:9",
  "videoArchiveFolderId": "[existing]",
  "userUploadedImagesFolderId": "[existing]"
}
```
**Remove:** `cloudinaryCloudName`, `cloudinaryApiKey` (no longer needed)

---

## Error Handling (Comprehensive)

### Three-Tier Error Strategy
1. **Retry on IN_PROGRESS** - Continue polling up to 30 attempts (10 min max)
2. **Log ALL failures** - Google Sheets log captures every error with full context
3. **Notify user** - Only after retries exhausted or explicit FAILED status

### Error Detection Points
| Status | Action |
|--------|--------|
| `COMPLETED` | Success - proceed to video download |
| `FAILED` | Immediate error - log + notify user |
| `IN_PROGRESS` after 30 retries | Timeout - log + notify user |
| HTTP error | Network issue - log + notify user |

### Google Sheets Error Log Schema
| Column | Value |
|--------|-------|
| timestamp | Execution time |
| workflow | Picture with Picasso v3.0 |
| error_type | API_FAILURE / TIMEOUT / NETWORK_ERROR |
| request_id | Kling request ID for debugging |
| retry_count | Number of polling attempts |
| user_source | telegram / form |
| user_id | Chat ID or email |

---

## Files to Update

| File | Action |
|------|--------|
| `source/workflows/Picture with Picasso -Wan 2.5-v2.4.json` | Rename to v3.0, modify (main workflow) |
| `versions/workflows/Picture with Picasso -Wan 2.5-v2.4.json` | Archive (backup before changes) |
| `docs/guides/Config-03-Reve Remix Configuration.md` | Move to `versions/prompts/` |
| `docs/guides/Config-04-Kling Reference-to-Video Configuration.md` | Create (new config guide) |
| `docs/design/WORKFLOW-REDESIGN-PLAN.md` | Update with new architecture |
| Google Sheets | Create "Workflow Error Log" spreadsheet with "Kling Errors" sheet |

---

## Implementation Steps

1. **Backup current workflow** - Copy v2.4.json to `versions/workflows/`
2. **Rename workflow** - Change name to "Picture with Picasso v3.0"
3. **Create Google Sheets error log** - Set up "Workflow Error Log" spreadsheet
4. **Remove Reve nodes** - Delete 6 Reve-related nodes
5. **Remove Kie.ai nodes** - Delete 4 video generation nodes
6. **Add Kling nodes** - Create 10 new nodes with configurations above
7. **Configure parallel archive** - Set up branching for parallel reference image archive
8. **Rewire connections** - Connect new nodes per flow diagram
9. **Update prompts** - Add combined prompt template to Prepare Kling Elements
10. **Add error handling** - Configure Google Sheets + notification nodes
11. **Test submission** - Verify API connectivity with FAL_KEY
12. **Test error handling** - Verify Google Sheets logging works
13. **Test end-to-end** - Run full workflow with test images
14. **Update documentation** - Create new Config-04 guide
15. **Archive old configs** - Move Reve config to versions/

---

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| Kling identity preservation may not match expectations | Test with various user/Picasso image combinations before finalizing |
| API response time may be longer than Reve + Kie.ai combined | Monitor and adjust wait duration if needed |
| FAL_KEY may need different permissions for Kling | Verify FAL account has Kling O1 Pro access |
| Base64 image size limits | Current 2048px resize should be sufficient |

---

## Summary

**This plan reconfigures the Picture with Picasso n8n workflow from:**
- Reve Remix (image composition) + Kie.ai Wan 2.5 (video generation)

**To:**
- Kling O1 Reference-to-Video (direct multi-identity video generation)

**Key Changes:**
- Remove 10 nodes (Reve + Kie.ai pipelines)
- Add 10 nodes (Kling API + error handling)
- 20-second polling interval (validated by n8n docs)
- Comprehensive error handling (retry + log + notify)
- Parallel reference image archiving
- Version bump to v3.0

**Ready for implementation upon approval.**

---

## Related Documents

### Design Configuration Files
- `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` - **Kling O1 node configurations (RECOMMENDED)**
- `docs/design/design-wan-2.6-configuration-v1.0-2026-01-06.md` - WAN 2.6 node configurations (alternative with limitations)

### Analysis Documents
- `docs/analysis/analysis-tool-comparative-analysis-v1.0-2026-01-06.md` - Tool comparison research

### Other References
- `docs/design/PICTURE-WITH-PICASSO-BRD.md` - Business requirements
- `docs/guides/Config-01-n8n to Reve Configuration.md` - Current Reve config
