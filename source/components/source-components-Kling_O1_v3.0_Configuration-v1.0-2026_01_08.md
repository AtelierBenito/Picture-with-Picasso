# Kling O1 v3.0 Configuration

**Version:** v1.0-2026-01-08
**Workflow Version:** Picture with Picasso v3.0
**API:** fal-ai/kling-video/o1/reference-to-video

---

## Overview

This document contains all configuration details for the Kling O1 integration in Picture with Picasso v3.0, including API endpoints, node configurations, and operational parameters.

---

## API Configuration

### Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video` | POST | Submit video generation request |
| `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/{request_id}/status` | GET | Check request status |
| `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/{request_id}` | GET | Get completed result |

### Authentication

| Header | Value | Source |
|--------|-------|--------|
| `Authorization` | `Key {FAL_KEY}` | Workflow Configuration node or environment variable |
| `Content-Type` | `application/json` | Static |

### Request Payload Schema

```json
{
  "prompt": "string - video generation prompt with @Element references",
  "elements": [
    {
      "reference_image_urls": ["string - base64 or URL"],
      "frontal_image_url": "string - base64 or URL"
    }
  ],
  "duration": "string - video duration in seconds",
  "aspect_ratio": "string - aspect ratio (e.g., '9:16')"
}
```

### Response Schemas

#### Submit Response
```json
{
  "request_id": "string - unique identifier for polling"
}
```

#### Status Response
```json
{
  "status": "IN_QUEUE | IN_PROGRESS | COMPLETED | FAILED"
}
```

#### Result Response
```json
{
  "video": {
    "url": "string - URL to download video file"
  }
}
```

---

## Video Generation Parameters

### Core Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `duration` | `10` | 10 seconds for meaningful interaction, social media native |
| `aspect_ratio` | `9:16` | Vertical format for TikTok, Instagram Reels, Stories |
| `negative_prompt` | See below | Quality control derived from v2.4 + best practices |

### Negative Prompt

```
face morphing, identity swap, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur
```

| Item | Source | Purpose |
|------|--------|---------|
| `face morphing` | Kling O1 best practices | Prevents gradual face changes during video |
| `identity swap` | User requirement | Keeps @Element1 and @Element2 distinct |
| `duplicated limbs` | v2.4 Reve prompt | Proven quality control from production |
| `extra limbs` | v2.4 Reve prompt | Reinforces limb integrity in multi-person scenes |
| `distorted anatomy` | v2.4 Reve prompt | Maintains realistic body proportions |
| `unnatural proportions` | v2.4 scaling reference | Maintains realistic sizing |
| `changing background` | User requirement | Environment consistency throughout video |
| `motion blur` | v2.4 Reve prompt | Video quality control |

### @Element References

| Reference | Source | Purpose |
|-----------|--------|---------|
| `@Element1` | User uploaded photo | Visitor identity - preserved throughout video |
| `@Element2` | Picasso reference image | Picasso identity + scene visual reference |

### Prompt Template

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

## Polling Configuration

### Timing Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `polling_interval` | 20 seconds | Under 65s threshold - no database offload (n8n docs) |
| `max_retries` | 30 | 30 × 20s = 10 minute maximum timeout |
| `timeout_ms` | 120000 | HTTP request timeout for initial submission |

### Status Values

| Status | Action | Next Node |
|--------|--------|-----------|
| `COMPLETED` | Fetch result | Get Kling Result |
| `FAILED` | Log error, notify user | Error handling |
| `IN_QUEUE` | Continue polling | Increment Retry Counter |
| `IN_PROGRESS` | Continue polling | Increment Retry Counter |

---

## Node Configurations

### Node 1: Prepare Kling Elements

**Type:** `n8n-nodes-base.set`
**typeVersion:** 3.4

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "kling-elements",
          "name": "elements",
          "value": "={{ [\n  {\n    \"reference_image_urls\": [$('Convert User Image to Base64').item.json.userImageBase64],\n    \"frontal_image_url\": $('Convert User Image to Base64').item.json.userImageBase64\n  },\n  {\n    \"reference_image_urls\": [$('Convert Picasso Image to Base64').item.json.picassoImageBase64],\n    \"frontal_image_url\": $('Convert Picasso Image to Base64').item.json.picassoImageBase64\n  }\n] }}",
          "type": "array"
        },
        {
          "id": "kling-prompt",
          "name": "prompt",
          "value": "[Full prompt from template above]",
          "type": "string"
        },
        {
          "id": "kling-duration",
          "name": "duration",
          "value": "10",
          "type": "string"
        },
        {
          "id": "kling-aspect-ratio",
          "name": "aspect_ratio",
          "value": "9:16",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": true
  }
}
```

**Data Flow:**
- Input: `userImageBase64`, `picassoImageBase64` from Base64 conversion nodes
- Output: `elements`, `prompt`, `duration`, `aspect_ratio`
- Preserves: `inputSource`, `chatId`, `userEmail` for routing

### Node 2: Submit to Kling

**Type:** `n8n-nodes-base.httpRequest`
**typeVersion:** 4.2

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "=Key {{ $('Workflow Configuration').item.json.falApiKey }}"
        },
        {
          "name": "Content-Type",
          "value": "application/json"
        }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({\n  \"prompt\": $json.prompt,\n  \"elements\": $json.elements,\n  \"duration\": $json.duration,\n  \"aspect_ratio\": $json.aspect_ratio\n}) }}",
    "options": {
      "timeout": 120000
    }
  }
}
```

**Data Flow:**
- Input: `prompt`, `elements`, `duration`, `aspect_ratio`
- Output: `request_id`

### Node 3: Initialize Retry Counter

**Type:** `n8n-nodes-base.set`
**typeVersion:** 3.4

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "retry-count",
          "name": "retryCount",
          "value": "0",
          "type": "number"
        },
        {
          "id": "request-id",
          "name": "request_id",
          "value": "={{ $json.request_id }}",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": true
  }
}
```

**Data Flow:**
- Input: `request_id` from Submit node
- Output: `retryCount = 0`, `request_id`

### Node 4: Check Kling Status

**Type:** `n8n-nodes-base.httpRequest`
**typeVersion:** 4.2

```json
{
  "parameters": {
    "method": "GET",
    "url": "={{ 'https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/' + $json.request_id + '/status' }}",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "=Key {{ $('Workflow Configuration').item.json.falApiKey }}"
        }
      ]
    }
  }
}
```

**Data Flow:**
- Input: `request_id`
- Output: `status`

### Node 5: Status Router

**Type:** `n8n-nodes-base.switch`
**typeVersion:** 3.2

```json
{
  "parameters": {
    "mode": "rules",
    "rules": {
      "values": [
        {
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "COMPLETED",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          },
          "renameOutput": true,
          "outputKey": "COMPLETED"
        },
        {
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "FAILED",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          },
          "renameOutput": true,
          "outputKey": "FAILED"
        },
        {
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "IN_QUEUE",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          },
          "renameOutput": true,
          "outputKey": "PENDING"
        },
        {
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "IN_PROGRESS",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          },
          "renameOutput": true,
          "outputKey": "PENDING"
        }
      ]
    },
    "options": {
      "fallbackOutput": "extra"
    }
  }
}
```

**Output Routing:**
| Output | Status | Next Node |
|--------|--------|-----------|
| 0 | COMPLETED | Get Kling Result |
| 1 | FAILED | Error handling |
| 2 | IN_QUEUE | Increment Retry Counter |
| 3 | IN_PROGRESS | Increment Retry Counter |

### Node 6: Get Kling Result

**Type:** `n8n-nodes-base.httpRequest`
**typeVersion:** 4.2

```json
{
  "parameters": {
    "method": "GET",
    "url": "={{ 'https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/' + $json.request_id }}",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "=Key {{ $('Workflow Configuration').item.json.falApiKey }}"
        }
      ]
    },
    "options": {
      "response": {
        "response": {
          "responseFormat": "json"
        }
      }
    }
  }
}
```

**Data Flow:**
- Input: `request_id`
- Output: `video.url`

### Node 7: Increment Retry Counter

**Type:** `n8n-nodes-base.set`
**typeVersion:** 3.4

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "increment-retry",
          "name": "retryCount",
          "value": "={{ $json.retryCount + 1 }}",
          "type": "number"
        }
      ]
    },
    "includeOtherFields": true
  }
}
```

### Node 8: Check Max Retries

**Type:** `n8n-nodes-base.if`
**typeVersion:** 2.2

```json
{
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "typeValidation": "strict"
      },
      "conditions": [
        {
          "leftValue": "={{ $json.retryCount }}",
          "rightValue": "30",
          "operator": { "type": "number", "operation": "gte" }
        }
      ]
    }
  }
}
```

**Output Routing:**
| Output | Condition | Next Node |
|--------|-----------|-----------|
| TRUE | retryCount >= 30 | Error handling (timeout) |
| FALSE | retryCount < 30 | Wait for Kling |

### Node 9: Wait for Kling

**Type:** `n8n-nodes-base.wait`
**typeVersion:** 1.1

```json
{
  "parameters": {
    "resume": "timeInterval",
    "amount": 20,
    "unit": "seconds"
  }
}
```

**Note:** 20 seconds is under the 65-second threshold where n8n doesn't offload execution data to the database.

### Node 10: Download Kling Video

**Type:** `n8n-nodes-base.httpRequest`
**typeVersion:** 4.2

```json
{
  "parameters": {
    "url": "={{ $('Get Kling Result').item.json.video.url }}",
    "options": {
      "response": {
        "response": {
          "responseFormat": "file",
          "outputPropertyName": "data"
        }
      }
    }
  }
}
```

**Data Flow:**
- Input: `video.url` from Get Kling Result
- Output: Binary video file in `data` property

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `FAL_KEY` | fal.ai API key | Yes |

**Alternative:** Store in `Workflow Configuration` node as `falApiKey`.

---

## Social Media Compatibility

| Platform | 9:16 Support | 10s Duration | Max Duration |
|----------|--------------|--------------|--------------|
| TikTok | Native | Native | 10 min |
| Instagram Reels | Native | Native | 90 sec |
| Instagram Stories | Native | Native (auto-split) | 60 sec |
| X (Twitter) | Supported | Supported | 2:20 |
| YouTube Shorts | Native | Native | 60 sec |

---

## Error Handling

### Error Types

| Error | Cause | Resolution |
|-------|-------|------------|
| `FAILED` status | API generation failure | Log to Google Sheets, notify user |
| Max retries exceeded | Timeout (>10 min) | Log to Google Sheets, notify user |
| HTTP 401 | Invalid FAL_KEY | Check API key configuration |
| HTTP 429 | Rate limit | Increase polling interval |
| HTTP 500 | fal.ai server error | Retry or notify user |

### Error Notification Template

```
Sorry, we encountered an issue generating your video with Picasso.
Our team has been notified. Please try again later.
Error Reference: {{ $json.request_id }}
```

---

## Data Flow Summary

```
Input Data Available:
├── userImageBase64 (from Convert User Image to Base64)
├── picassoImageBase64 (from Convert Picasso Image to Base64)
├── inputSource ("telegram" or "form")
├── chatId (if Telegram)
└── userEmail (if form)

Output Data Produced:
├── video binary (for Archive Video to Google Drive)
├── video.url (for email/telegram delivery)
└── All input data preserved (for routing)
```

---

## Related Documents

| Document | Location |
|----------|----------|
| Finalized Prompt | `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.0-2026_01_08.md` |
| Node Insert JSON | `source/workflows/source-workflows-Kling_O1_Nodes_Insert-v1.0-2026_01_08.json` |
| Changelog | `docs/changelogs/docs-changelogs-Changelog_v2.4_to_v3.0-v1.0-2026_01_08.md` |
| Implementation Guide | `docs/guides/docs-guides-Kling_O1_v3.0_Implementation_Guide-v1.0-2026_01_08.md` |
| Background Research | `docs/research/docs-research-Kling_O1_Background_Analysis-v1.0-2026_01_08.md` |

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-08 | v1.0 | Initial configuration document for v3.0 |
