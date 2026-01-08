# Kling O1 Reference-to-Video Configuration

**Version:** v1.0-2026-01-06
**API Provider:** fal.ai
**Endpoint:** `fal-ai/kling-video/o1/reference-to-video`
**Purpose:** Multi-identity video generation with audio/interaction support

---

## Use Case Requirements

| Requirement | Support |
|-------------|---------|
| Two input images | Yes |
| Multiple people per image | Yes (up to 7 total references) |
| Identity preservation | Yes (Elements system) |
| Talking/banter | Yes (native audio generation) |
| Background noises | Yes (via prompt) |
| Naturalistic interaction | Yes (via prompt engineering) |

---

## Workflow Insertion Point

### INBOUND Connection (Before Kling)

```
EXISTING WORKFLOW                          NEW KLING NODES
================                          ================

Convert User Image to Base64 ─────────┐
                                       │
                                       ├──→ [Prepare Kling Elements]
                                       │
Convert Picasso Image to Base64 ──────┘
```

**Connects FROM:**
- `Convert User Image to Base64` (output: `userImageBase64`)
- `Convert Picasso Image to Base64` (output: `picassoImageBase64`)

### OUTBOUND Connection (After Kling)

```
NEW KLING NODES                           EXISTING WORKFLOW
===============                           ================

[Get Kling Result] ──────────────────────→ Archive Video to Google Drive
        │                                           │
        │                                           ↓
        │                                      Route Output
        │                                       /        \
        │                                 Telegram      Email
        │
        └─→ (video.url output)
```

**Connects TO:**
- `Archive Video to Google Drive` (input: video URL from Kling result)

---

## Complete Node Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           KLING O1 NODE CHAIN                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [INBOUND]                                                                  │
│      │                                                                      │
│      ↓                                                                      │
│  ┌──────────────────────────┐                                              │
│  │ 1. Prepare Kling Elements │ ← Structures elements array + prompt        │
│  └──────────────────────────┘                                              │
│      │                                                                      │
│      ↓                                                                      │
│  ┌──────────────────────────┐                                              │
│  │ 2. Submit to Kling       │ ← POST to fal.ai queue                       │
│  └──────────────────────────┘                                              │
│      │                                                                      │
│      ↓                                                                      │
│  ┌──────────────────────────┐                                              │
│  │ 3. Initialize Retry      │ ← Set retryCount = 0                         │
│  └──────────────────────────┘                                              │
│      │                                                                      │
│      ↓                                                                      │
│  ┌──────────────────────────┐      ┌─────────────────┐                     │
│  │ 4. Check Kling Status    │ ←────│ 8. Wait 20s     │                     │
│  └──────────────────────────┘      └─────────────────┘                     │
│      │                                     ↑                                │
│      ↓                                     │                                │
│  ┌──────────────────────────┐              │                                │
│  │ 5. Status Router         │──PENDING────→│                                │
│  │    (Switch node)         │              │                                │
│  └──────────────────────────┘              │                                │
│      │         │                           │                                │
│  COMPLETED   FAILED                        │                                │
│      │         │         ┌─────────────────┘                                │
│      ↓         ↓         │                                                  │
│  ┌────────┐ ┌────────┐ ┌─────────────────┐                                 │
│  │6. Get  │ │9. Log  │ │7. Increment     │                                 │
│  │Result  │ │Error   │ │   Retry Counter │                                 │
│  └────────┘ └────────┘ └─────────────────┘                                 │
│      │         │              │                                             │
│      │         ↓              ↓                                             │
│      │    ┌────────┐   ┌─────────────────┐                                 │
│      │    │10.Notify│  │7b. Check Max    │──YES→ [9. Log Error]            │
│      │    │  User  │   │    Retries (30) │                                 │
│      │    └────────┘   └─────────────────┘                                 │
│      │                        │ NO                                          │
│      │                        ↓                                             │
│      │                 [8. Wait 20s] ────→ [4. Check Status]               │
│      │                                                                      │
│      ↓                                                                      │
│  [OUTBOUND]                                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Node Configurations

### Node 1: Prepare Kling Elements

**Name:** `Prepare Kling Elements`
**Type:** `n8n-nodes-base.set`
**Position:** After Base64 conversion nodes

**Configuration:**
```json
{
  "mode": "manual",
  "duplicateItem": false,
  "assignments": {
    "assignments": [
      {
        "id": "elements",
        "name": "elements",
        "value": "={{ [\n  {\n    \"reference_image_urls\": [$('Convert User Image to Base64').item.json.userImageBase64],\n    \"frontal_image_url\": $('Convert User Image to Base64').item.json.userImageBase64\n  },\n  {\n    \"reference_image_urls\": [$('Convert Picasso Image to Base64').item.json.picassoImageBase64],\n    \"frontal_image_url\": $('Convert Picasso Image to Base64').item.json.picassoImageBase64\n  }\n] }}",
        "type": "array"
      },
      {
        "id": "prompt",
        "name": "prompt",
        "value": "={{ $json.systemPrompt || `Create an animated scene where @Element1 (the visitors) meet @Element2 (Picasso) in his art studio.\n\nScene Setup:\n- Setting: Picasso's famous studio with cubist paintings on the walls, art supplies scattered around, warm Mediterranean light\n- @Element1 contains the visiting people who should maintain their exact appearances and identities\n- @Element2 is Picasso who should look authentic to his iconic appearance\n\nInteraction Requirements:\n- Natural conversation with talking, gestures, and expressions\n- Friendly banter and laughter between the subjects\n- Picasso showing his artwork or demonstrating painting techniques\n- Background ambient sounds: studio atmosphere, perhaps distant street sounds, brush strokes\n- Warm, welcoming body language from all participants\n- Eye contact and engaged conversation\n\nAnimation Style:\n- Cinematic camera movements\n- Natural lip movements synchronized with implied dialogue\n- Expressive hand gestures\n- Authentic period-appropriate setting\n\nTechnical:\n- Maintain stable identity for all people throughout\n- 16:9 aspect ratio\n- Smooth, professional motion` }}",
        "type": "string"
      },
      {
        "id": "duration",
        "name": "duration",
        "value": "={{ $('Workflow Configuration').item.json.duration || '5' }}",
        "type": "string"
      },
      {
        "id": "aspect_ratio",
        "name": "aspect_ratio",
        "value": "16:9",
        "type": "string"
      },
      {
        "id": "request_id",
        "name": "request_id",
        "value": "",
        "type": "string"
      }
    ]
  },
  "options": {}
}
```

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `elements` | Array | Array of element objects with reference images |
| `prompt` | String | Combined scene/animation prompt with @Element references |
| `duration` | String | "5" or "10" seconds |
| `aspect_ratio` | String | "16:9", "9:16", or "1:1" |

---

### Node 2: Submit to Kling

**Name:** `Submit to Kling`
**Type:** `n8n-nodes-base.httpRequest`
**Position:** After Prepare Kling Elements

**Configuration:**
```json
{
  "method": "POST",
  "url": "https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {
        "name": "Authorization",
        "value": "Key {{ $env.FAL_KEY }}"
      },
      {
        "name": "Content-Type",
        "value": "application/json"
      }
    ]
  },
  "sendBody": true,
  "bodyParameters": {
    "parameters": []
  },
  "specifyBody": "json",
  "jsonBody": "={{ JSON.stringify({\n  \"prompt\": $json.prompt,\n  \"elements\": $json.elements,\n  \"duration\": $json.duration,\n  \"aspect_ratio\": $json.aspect_ratio\n}) }}",
  "options": {
    "response": {
      "response": {
        "fullResponse": false,
        "responseFormat": "json"
      }
    }
  }
}
```

**Request Body Schema:**
```json
{
  "prompt": "Create an animated scene where @Element1...",
  "elements": [
    {
      "reference_image_urls": ["data:image/png;base64,..."],
      "frontal_image_url": "data:image/png;base64,..."
    },
    {
      "reference_image_urls": ["data:image/png;base64,..."],
      "frontal_image_url": "data:image/png;base64,..."
    }
  ],
  "duration": "5",
  "aspect_ratio": "16:9"
}
```

**Response:**
```json
{
  "request_id": "abc123-def456-ghi789"
}
```

---

### Node 3: Initialize Retry Counter

**Name:** `Initialize Retry Counter`
**Type:** `n8n-nodes-base.set`
**Position:** After Submit to Kling

**Configuration:**
```json
{
  "mode": "manual",
  "duplicateItem": false,
  "assignments": {
    "assignments": [
      {
        "id": "retryCount",
        "name": "retryCount",
        "value": "0",
        "type": "number"
      },
      {
        "id": "request_id",
        "name": "request_id",
        "value": "={{ $json.request_id }}",
        "type": "string"
      }
    ]
  },
  "options": {
    "includeBinary": true
  }
}
```

---

### Node 4: Check Kling Status

**Name:** `Check Kling Status`
**Type:** `n8n-nodes-base.httpRequest`
**Position:** After Initialize Retry Counter (and loops back from Wait)

**Configuration:**
```json
{
  "method": "GET",
  "url": "={{ 'https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/' + $json.request_id + '/status' }}",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {
        "name": "Authorization",
        "value": "Key {{ $env.FAL_KEY }}"
      }
    ]
  },
  "options": {
    "response": {
      "response": {
        "fullResponse": false,
        "responseFormat": "json"
      }
    }
  }
}
```

**Response:**
```json
{
  "status": "IN_QUEUE" | "IN_PROGRESS" | "COMPLETED" | "FAILED",
  "request_id": "abc123-def456-ghi789"
}
```

---

### Node 5: Status Router

**Name:** `Status Router`
**Type:** `n8n-nodes-base.switch`
**Position:** After Check Kling Status

**Configuration:**
```json
{
  "mode": "rules",
  "output": "all",
  "rules": {
    "values": [
      {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "leftValue": "={{ $json.status }}",
              "rightValue": "COMPLETED",
              "operator": {
                "type": "string",
                "operation": "equals"
              }
            }
          ],
          "combinator": "and"
        },
        "renameOutput": true,
        "outputKey": "COMPLETED"
      },
      {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "leftValue": "={{ $json.status }}",
              "rightValue": "FAILED",
              "operator": {
                "type": "string",
                "operation": "equals"
              }
            }
          ],
          "combinator": "and"
        },
        "renameOutput": true,
        "outputKey": "FAILED"
      },
      {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "leftValue": "={{ $json.status }}",
              "rightValue": "IN_QUEUE",
              "operator": {
                "type": "string",
                "operation": "equals"
              }
            }
          ],
          "combinator": "and"
        },
        "renameOutput": true,
        "outputKey": "PENDING"
      },
      {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "leftValue": "={{ $json.status }}",
              "rightValue": "IN_PROGRESS",
              "operator": {
                "type": "string",
                "operation": "equals"
              }
            }
          ],
          "combinator": "and"
        },
        "renameOutput": true,
        "outputKey": "PENDING"
      }
    ]
  },
  "options": {}
}
```

**Outputs:**
| Output | Route To |
|--------|----------|
| COMPLETED | Get Kling Result |
| FAILED | Log Error to Google Sheets |
| PENDING | Increment Retry Counter |

---

### Node 6: Get Kling Result

**Name:** `Get Kling Result`
**Type:** `n8n-nodes-base.httpRequest`
**Position:** After Status Router (COMPLETED branch)

**Configuration:**
```json
{
  "method": "GET",
  "url": "={{ 'https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video/requests/' + $json.request_id }}",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {
        "name": "Authorization",
        "value": "Key {{ $env.FAL_KEY }}"
      }
    ]
  },
  "options": {
    "response": {
      "response": {
        "fullResponse": false,
        "responseFormat": "json"
      }
    }
  }
}
```

**Response:**
```json
{
  "video": {
    "url": "https://v3b.fal.media/files/..../output.mp4",
    "content_type": "video/mp4",
    "file_name": "output.mp4",
    "file_size": 47359974
  }
}
```

**Output Mapping:**
| Field | Path | Use |
|-------|------|-----|
| Video URL | `$json.video.url` | Pass to Archive Video node |
| File Size | `$json.video.file_size` | Logging |
| Content Type | `$json.video.content_type` | Validation |

---

### Node 7: Increment Retry Counter

**Name:** `Increment Retry Counter`
**Type:** `n8n-nodes-base.set`
**Position:** After Status Router (PENDING branch)

**Configuration:**
```json
{
  "mode": "manual",
  "duplicateItem": false,
  "assignments": {
    "assignments": [
      {
        "id": "retryCount",
        "name": "retryCount",
        "value": "={{ $json.retryCount + 1 }}",
        "type": "number"
      }
    ]
  },
  "options": {
    "includeBinary": true
  }
}
```

---

### Node 7b: Check Max Retries

**Name:** `Check Max Retries`
**Type:** `n8n-nodes-base.if`
**Position:** After Increment Retry Counter

**Configuration:**
```json
{
  "conditions": {
    "options": {
      "caseSensitive": true,
      "leftValue": "",
      "typeValidation": "strict"
    },
    "conditions": [
      {
        "id": "maxRetries",
        "leftValue": "={{ $json.retryCount }}",
        "rightValue": "30",
        "operator": {
          "type": "number",
          "operation": "gte"
        }
      }
    ],
    "combinator": "and"
  },
  "options": {}
}
```

**Outputs:**
| Output | Condition | Route To |
|--------|-----------|----------|
| TRUE | retryCount >= 30 | Log Error to Google Sheets |
| FALSE | retryCount < 30 | Wait for Kling |

---

### Node 8: Wait for Kling

**Name:** `Wait for Kling`
**Type:** `n8n-nodes-base.wait`
**Position:** After Check Max Retries (FALSE branch)

**Configuration:**
```json
{
  "resume": "timeInterval",
  "unit": "seconds",
  "amount": 20
}
```

**Note:** 20 seconds is under the 65-second threshold, so n8n keeps execution in memory (no database offload).

**Connection:** Loops back to `Check Kling Status`

---

### Node 9: Log Error to Google Sheets

**Name:** `Log Error to Google Sheets`
**Type:** `n8n-nodes-base.googleSheets`
**Position:** After Status Router (FAILED) or Check Max Retries (TRUE)

**Configuration:**
```json
{
  "operation": "appendOrUpdate",
  "documentId": {
    "__rl": true,
    "value": "[YOUR_SPREADSHEET_ID]",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "Kling Errors",
    "mode": "name"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "value": {
      "timestamp": "={{ $now.toISO() }}",
      "workflow": "Picture with Picasso v3.0",
      "api": "Kling O1",
      "error_type": "={{ $json.status === 'FAILED' ? 'API_FAILURE' : 'TIMEOUT' }}",
      "request_id": "={{ $json.request_id }}",
      "retry_count": "={{ $json.retryCount }}",
      "user_source": "={{ $('Workflow Configuration').item.json.inputSource }}",
      "user_id": "={{ $('Workflow Configuration').item.json.inputSource === 'telegram' ? $('Workflow Configuration').item.json.chatId : $('Workflow Configuration').item.json.userEmail }}"
    }
  },
  "options": {}
}
```

**Spreadsheet Schema:**
| Column | Type | Description |
|--------|------|-------------|
| timestamp | DateTime | ISO timestamp |
| workflow | String | "Picture with Picasso v3.0" |
| api | String | "Kling O1" |
| error_type | String | "API_FAILURE" or "TIMEOUT" |
| request_id | String | fal.ai request ID |
| retry_count | Number | Number of polling attempts |
| user_source | String | "telegram" or "form" |
| user_id | String | Chat ID or email |

---

### Node 10: Notify User of Failure

**Name:** `Notify User of Failure`
**Type:** `n8n-nodes-base.if`
**Position:** After Log Error to Google Sheets

**Configuration:**
```json
{
  "conditions": {
    "options": {
      "caseSensitive": true,
      "leftValue": "",
      "typeValidation": "strict"
    },
    "conditions": [
      {
        "id": "isTelegram",
        "leftValue": "={{ $('Workflow Configuration').item.json.inputSource }}",
        "rightValue": "telegram",
        "operator": {
          "type": "string",
          "operation": "equals"
        }
      }
    ],
    "combinator": "and"
  },
  "options": {}
}
```

**TRUE Branch:** Connect to Telegram Send Message node
**FALSE Branch:** Connect to Gmail Send Email node

**Error Message Template:**
```
Sorry, we encountered an issue generating your video with Picasso.

Our team has been notified and is looking into it.
Please try again in a few minutes.

Error Reference: {{ $json.request_id }}
```

---

## Environment Variables Required

| Variable | Description | Example |
|----------|-------------|---------|
| `FAL_KEY` | fal.ai API key | `fal_xxxxxxxxxxxx` |

---

## Prompt Engineering for Audio/Interaction

### Enabling Natural Audio

Kling O1 supports native audio generation. Include these elements in your prompt:

```
Interaction Requirements:
- Natural conversation with talking, gestures, and expressions
- Friendly banter and laughter between the subjects
- [Character] speaking about [topic]
- Background ambient sounds: [environment sounds]
- Lip movements synchronized with implied dialogue
```

### Multi-Person Prompt Structure

For images with multiple people:

```
@Element1 contains [NUMBER] people: [DESCRIPTION of each person].
Each person should maintain their distinct appearance and identity.

@Element2 is Picasso who should look authentic to his iconic appearance.

The group from @Element1 interacts naturally with Picasso:
- Person A asks about [topic]
- Person B laughs at a joke
- Picasso gestures enthusiastically while explaining
```

---

## API Limits and Considerations

| Limit | Value |
|-------|-------|
| Max elements | 7 total (elements + reference images) |
| Max duration | 10 seconds |
| Supported aspect ratios | 16:9, 9:16, 1:1 |
| Image format | Base64 data URI or hosted URL |
| Max image size | Recommend 2048px max dimension |

---

## Cost Estimate

| Duration | Estimated Cost |
|----------|----------------|
| 5 seconds | ~$0.15-0.25 |
| 10 seconds | ~$0.30-0.50 |

---

## Related Documents

- `docs/design/design-wan-2.6-configuration-v1.0-2026-01-06.md` - Alternative: WAN 2.6 configuration
- `docs/analysis/analysis-workflow-reconfiguration-plan-v1.0-2026-01-06.md` - Implementation plan
- `docs/analysis/analysis-tool-comparative-analysis-v1.0-2026-01-06.md` - Tool comparison
