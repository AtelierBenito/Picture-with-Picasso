# WAN 2.6 Configuration Guide - Picture with Picasso

**Version:** v1.0-2026-01-06
**Status:** Alternative Option (with limitations)
**API Provider:** fal.ai
**Workflow Target:** v3.0

---

## Critical Limitation Notice

**WAN 2.6 does NOT directly support multi-identity image-based video generation.**

| User Requirement | WAN 2.6 Support |
|------------------|-----------------|
| Two input IMAGES | **NOT SUPPORTED** - R2V requires VIDEO inputs |
| Multi-person identity preservation | Partial - only via video references |
| Direct image-to-video with multiple identities | **NOT AVAILABLE** |

### Why This Matters

The user's workflow requires:
1. Two INPUT IMAGES (user photo + Picasso photo)
2. Both identities preserved exactly
3. Direct video generation

**WAN 2.6 Endpoints Available:**
- `wan/v2.6/image-to-video` - Single image only, no multi-identity
- `wan/v2.6/reference-to-video` - Requires VIDEO references (not images)

**Recommendation:** Use **Kling O1 Reference-to-Video** as the primary solution. WAN 2.6 can be a fallback only with the two-phase workaround described below.

---

## Two-Phase Workaround (If Kling Fails)

If Kling O1 identity preservation is insufficient, WAN 2.6 can be used with this two-phase approach:

```
Phase 1: Create Reference Videos
User Image → WAN Image-to-Video → 5s User Video (reference)
Picasso Image → WAN Image-to-Video → 5s Picasso Video (reference)

Phase 2: Combine with Reference-to-Video
User Video + Picasso Video → WAN Reference-to-Video → Final Video
```

**Trade-offs:**
- 3x API calls instead of 1
- Higher cost (~$0.75 vs ~$0.25)
- Longer processing time
- Identity may drift in Phase 1

---

## API Endpoints

### Endpoint 1: Image-to-Video (Phase 1)

| Attribute | Value |
|-----------|-------|
| **Model ID** | `wan/v2.6/image-to-video` |
| **Queue URL** | `https://queue.fal.run/wan/v2.6/image-to-video` |
| **Status URL** | `https://queue.fal.run/wan/v2.6/image-to-video/requests/{request_id}/status` |
| **Result URL** | `https://queue.fal.run/wan/v2.6/image-to-video/requests/{request_id}` |
| **Authentication** | `Authorization: Key {FAL_KEY}` |
| **Input** | Single image URL |
| **Output** | Video URL |

### Endpoint 2: Reference-to-Video (Phase 2)

| Attribute | Value |
|-----------|-------|
| **Model ID** | `wan/v2.6/reference-to-video` |
| **Queue URL** | `https://queue.fal.run/wan/v2.6/reference-to-video` |
| **Status URL** | `https://queue.fal.run/wan/v2.6/reference-to-video/requests/{request_id}/status` |
| **Result URL** | `https://queue.fal.run/wan/v2.6/reference-to-video/requests/{request_id}` |
| **Authentication** | `Authorization: Key {FAL_KEY}` |
| **Input** | 1-3 video URLs |
| **Output** | Video URL |

---

## Input Schema Reference

### Image-to-Video Input

```json
{
  "prompt": "string (required, max 800 chars)",
  "image_url": "string (required) - URL or base64 data URI",
  "audio_url": "string (optional) - background audio",
  "resolution": "720p | 1080p (default: 1080p)",
  "duration": "5 | 10 | 15 (default: 5)",
  "negative_prompt": "string (optional, max 500 chars)",
  "enable_prompt_expansion": "boolean (default: true)",
  "multi_shots": "boolean (optional)",
  "seed": "integer (optional)",
  "enable_safety_checker": "boolean (default: true)"
}
```

### Reference-to-Video Input

```json
{
  "prompt": "string (required, max 800 chars) - use @Video1, @Video2, @Video3",
  "video_urls": ["string"] (required) - 1-3 video URLs,
  "aspect_ratio": "16:9 | 9:16 | 1:1 | 4:3 | 3:4 (default: 16:9)",
  "resolution": "720p | 1080p (default: 1080p)",
  "duration": "5 | 10 (default: 5) - NO 15s support for R2V",
  "negative_prompt": "string (optional, max 500 chars)",
  "enable_prompt_expansion": "boolean (default: true)",
  "multi_shots": "boolean (default: true)",
  "seed": "integer (optional)",
  "enable_safety_checker": "boolean (default: true)"
}
```

### Output Schema

```json
{
  "video": {
    "url": "string - download URL",
    "content_type": "video/mp4",
    "width": "integer",
    "height": "integer",
    "fps": "float",
    "duration": "float"
  },
  "seed": "integer",
  "actual_prompt": "string (if prompt expansion enabled)"
}
```

---

## Prompt Syntax

### Image-to-Video Prompts
Standard descriptive prompts. No special syntax needed.

### Reference-to-Video Prompts
Use `@Video1`, `@Video2`, `@Video3` to reference subjects from videos.

**Example:**
```
@Video1 (the user) and @Video2 (Picasso) are having a warm, friendly conversation in an artistic studio. @Video1 laughs at something @Video2 says while gesturing enthusiastically.
```

**Multi-Shot Syntax:**
```
[0-3s] @Video1 and @Video2 meet in the studio. [3-6s] They shake hands warmly. [6-10s] Both sit down and begin chatting animatedly.
```

---

## Node Configurations (Two-Phase Workflow)

### Inbound Connection Points

| Connection From | Purpose |
|----------------|---------|
| `Convert User Image to Base64` | User image for Phase 1a |
| `Convert Picasso Image to Base64` | Picasso image for Phase 1b |

### Outbound Connection Points

| Connection To | Purpose |
|--------------|---------|
| `Archive Video to Google Drive` | Save generated video |
| `Route Output` | Send to Telegram/Email |

---

## Phase 1a: Create User Reference Video

### Node 1a-1: Prepare User Video Request
**Type:** n8n-nodes-base.set (v3.4)
**Purpose:** Prepare Image-to-Video request for user image

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "user_image_url",
          "name": "user_image_url",
          "value": "={{ $('Convert User Image to Base64').item.json.userImageBase64 }}",
          "type": "string"
        },
        {
          "id": "user_prompt",
          "name": "user_prompt",
          "value": "A person stands naturally, looking at the camera with a friendly expression. Subtle head movements and natural blinking. Warm, soft lighting.",
          "type": "string"
        },
        {
          "id": "duration",
          "name": "duration",
          "value": "5",
          "type": "string"
        },
        {
          "id": "resolution",
          "name": "resolution",
          "value": "1080p",
          "type": "string"
        }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [920, 300],
  "name": "Prepare User Video Request"
}
```

### Node 1a-2: Submit User Image-to-Video
**Type:** n8n-nodes-base.httpRequest (v4.3)
**Purpose:** Submit user image to WAN I2V API

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://queue.fal.run/wan/v2.6/image-to-video",
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
    "specifyBody": "json",
    "jsonBody": "={\n  \"prompt\": \"{{ $json.user_prompt }}\",\n  \"image_url\": \"{{ $json.user_image_url }}\",\n  \"duration\": \"{{ $json.duration }}\",\n  \"resolution\": \"{{ $json.resolution }}\",\n  \"negative_prompt\": \"low resolution, blurry, distorted face, unnatural movements\",\n  \"enable_prompt_expansion\": true,\n  \"enable_safety_checker\": true\n}",
    "options": {
      "response": {
        "response": {
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1120, 300],
  "name": "Submit User Image-to-Video"
}
```

---

## Phase 1b: Create Picasso Reference Video

### Node 1b-1: Prepare Picasso Video Request
**Type:** n8n-nodes-base.set (v3.4)
**Purpose:** Prepare Image-to-Video request for Picasso image

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "picasso_image_url",
          "name": "picasso_image_url",
          "value": "={{ $('Convert Picasso Image to Base64').item.json.picassoImageBase64 }}",
          "type": "string"
        },
        {
          "id": "picasso_prompt",
          "name": "picasso_prompt",
          "value": "Picasso in his artistic studio, looking contemplative with subtle movements. His iconic features visible clearly. Artistic lighting with warm tones.",
          "type": "string"
        },
        {
          "id": "duration",
          "name": "duration",
          "value": "5",
          "type": "string"
        },
        {
          "id": "resolution",
          "name": "resolution",
          "value": "1080p",
          "type": "string"
        }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [920, 500],
  "name": "Prepare Picasso Video Request"
}
```

### Node 1b-2: Submit Picasso Image-to-Video
**Type:** n8n-nodes-base.httpRequest (v4.3)
**Purpose:** Submit Picasso image to WAN I2V API

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://queue.fal.run/wan/v2.6/image-to-video",
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
    "specifyBody": "json",
    "jsonBody": "={\n  \"prompt\": \"{{ $json.picasso_prompt }}\",\n  \"image_url\": \"{{ $json.picasso_image_url }}\",\n  \"duration\": \"{{ $json.duration }}\",\n  \"resolution\": \"{{ $json.resolution }}\",\n  \"negative_prompt\": \"low resolution, blurry, distorted face, unnatural movements\",\n  \"enable_prompt_expansion\": true,\n  \"enable_safety_checker\": true\n}",
    "options": {
      "response": {
        "response": {
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1120, 500],
  "name": "Submit Picasso Image-to-Video"
}
```

---

## Phase 1 Polling (Both Videos)

### Node 1-3: Wait for Phase 1
**Type:** n8n-nodes-base.wait (v1.0)
**Purpose:** Wait for both I2V requests to process

```json
{
  "parameters": {
    "amount": 20,
    "unit": "seconds"
  },
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1,
  "position": [1320, 400],
  "name": "Wait for Phase 1"
}
```

### Node 1-4: Check User Video Status
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/image-to-video/requests/{{ $('Submit User Image-to-Video').item.json.request_id }}/status",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1520, 300],
  "name": "Check User Video Status"
}
```

### Node 1-5: Check Picasso Video Status
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/image-to-video/requests/{{ $('Submit Picasso Image-to-Video').item.json.request_id }}/status",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1520, 500],
  "name": "Check Picasso Video Status"
}
```

### Node 1-6: Both Videos Complete?
**Type:** n8n-nodes-base.if (v2.2)
**Purpose:** Check if both Phase 1 videos are ready

```json
{
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "leftValue": "",
        "typeValidation": "strict"
      },
      "conditions": [
        {
          "id": "user_complete",
          "leftValue": "={{ $('Check User Video Status').item.json.status }}",
          "rightValue": "COMPLETED",
          "operator": {
            "type": "string",
            "operation": "equals"
          }
        },
        {
          "id": "picasso_complete",
          "leftValue": "={{ $('Check Picasso Video Status').item.json.status }}",
          "rightValue": "COMPLETED",
          "operator": {
            "type": "string",
            "operation": "equals"
          }
        }
      ],
      "combinator": "and"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.2,
  "position": [1720, 400],
  "name": "Both Videos Complete?"
}
```

### Node 1-7: Get User Video Result
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/image-to-video/requests/{{ $('Submit User Image-to-Video').item.json.request_id }}",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1920, 300],
  "name": "Get User Video Result"
}
```

### Node 1-8: Get Picasso Video Result
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/image-to-video/requests/{{ $('Submit Picasso Image-to-Video').item.json.request_id }}",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [1920, 500],
  "name": "Get Picasso Video Result"
}
```

---

## Phase 2: Reference-to-Video (Combine Videos)

### Node 2-1: Prepare Reference-to-Video Request
**Type:** n8n-nodes-base.set (v3.4)
**Purpose:** Prepare R2V request with both video references

```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "video_urls",
          "name": "video_urls",
          "value": "=[\"{{ $('Get User Video Result').item.json.video.url }}\", \"{{ $('Get Picasso Video Result').item.json.video.url }}\"]",
          "type": "string"
        },
        {
          "id": "prompt",
          "name": "prompt",
          "value": "@Video1 (the user) and @Video2 (Picasso) are having a warm, friendly interaction in a Picasso-style artistic studio.\n\n[0-3s] @Video1 and @Video2 meet in the studio, both smiling warmly.\n[3-6s] They begin an animated conversation, @Video1 gesturing enthusiastically while @Video2 listens with interest.\n[6-10s] Both laugh together at a shared joke, showing genuine connection and joy.\n\nMaintain the artistic, cubist-inspired atmosphere throughout. Both subjects should appear as close friends enjoying each other's company.",
          "type": "string"
        },
        {
          "id": "duration",
          "name": "duration",
          "value": "10",
          "type": "string"
        },
        {
          "id": "resolution",
          "name": "resolution",
          "value": "1080p",
          "type": "string"
        },
        {
          "id": "aspect_ratio",
          "name": "aspect_ratio",
          "value": "16:9",
          "type": "string"
        }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [2120, 400],
  "name": "Prepare Reference-to-Video Request"
}
```

### Node 2-2: Submit Reference-to-Video
**Type:** n8n-nodes-base.httpRequest (v4.3)
**Purpose:** Submit combined video request to WAN R2V API

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://queue.fal.run/wan/v2.6/reference-to-video",
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
    "specifyBody": "json",
    "jsonBody": "={\n  \"prompt\": \"{{ $json.prompt }}\",\n  \"video_urls\": {{ $json.video_urls }},\n  \"duration\": \"{{ $json.duration }}\",\n  \"resolution\": \"{{ $json.resolution }}\",\n  \"aspect_ratio\": \"{{ $json.aspect_ratio }}\",\n  \"negative_prompt\": \"low resolution, blurry, distorted faces, unnatural movements, identity drift\",\n  \"enable_prompt_expansion\": true,\n  \"multi_shots\": true,\n  \"enable_safety_checker\": true\n}",
    "options": {
      "response": {
        "response": {
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [2320, 400],
  "name": "Submit Reference-to-Video"
}
```

### Node 2-3: Check R2V Status
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/reference-to-video/requests/{{ $json.request_id }}/status",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [2720, 400],
  "name": "Check R2V Status"
}
```

### Node 2-4: R2V Complete?
**Type:** n8n-nodes-base.switch (v3.4)
**Purpose:** Route based on R2V status

```json
{
  "parameters": {
    "mode": "rules",
    "rules": {
      "rules": [
        {
          "value": "COMPLETED",
          "outputKey": "completed",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "COMPLETED",
                "operator": {
                  "type": "string",
                  "operation": "equals"
                }
              }
            ]
          }
        },
        {
          "value": "FAILED",
          "outputKey": "failed",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "FAILED",
                "operator": {
                  "type": "string",
                  "operation": "equals"
                }
              }
            ]
          }
        },
        {
          "value": "IN_PROGRESS",
          "outputKey": "in_progress",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "IN_PROGRESS",
                "operator": {
                  "type": "string",
                  "operation": "equals"
                }
              }
            ]
          }
        },
        {
          "value": "IN_QUEUE",
          "outputKey": "in_queue",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.status }}",
                "rightValue": "IN_QUEUE",
                "operator": {
                  "type": "string",
                  "operation": "equals"
                }
              }
            ]
          }
        }
      ],
      "fallbackOutput": "none"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3.4,
  "position": [2920, 400],
  "name": "R2V Complete?"
}
```

### Node 2-5: Wait for R2V
**Type:** n8n-nodes-base.wait (v1.0)

```json
{
  "parameters": {
    "amount": 20,
    "unit": "seconds"
  },
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1,
  "position": [3120, 500],
  "name": "Wait for R2V"
}
```

### Node 2-6: Get R2V Result
**Type:** n8n-nodes-base.httpRequest (v4.3)

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://queue.fal.run/wan/v2.6/reference-to-video/requests/{{ $('Submit Reference-to-Video').item.json.request_id }}",
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
          "responseFormat": "json"
        }
      }
    }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [3120, 300],
  "name": "Get R2V Result"
}
```

---

## Connection Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 1: Create Reference Videos                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  Convert User Image ──→ Prepare User Request ──→ Submit User I2V ──┐            │
│       to Base64                                                      │            │
│                                                                      ├──→ Wait   │
│  Convert Picasso    ──→ Prepare Picasso      ──→ Submit Picasso  ──┘    for     │
│  Image to Base64         Request                  I2V                    Phase 1  │
│                                                                             │      │
│                                                                             ↓      │
│  ┌──────────────────────────────────────────────────────────────────────────┘      │
│  │                                                                                  │
│  ↓                                                                                  │
│  Check User Status + Check Picasso Status ──→ Both Complete? ──┐                   │
│                                                   │              │                   │
│                                           (NO) ←─┘              │ (YES)             │
│                                             │                   ↓                   │
│                                             └───→ Wait ───→ Loop Back               │
│                                                                                      │
│                                                   ↓                                  │
│                             Get User Video Result + Get Picasso Video Result        │
│                                                   │                                  │
├───────────────────────────────────────────────────┼──────────────────────────────────┤
│                           PHASE 2: Combine Videos                                   │
├───────────────────────────────────────────────────┼──────────────────────────────────┤
│                                                   ↓                                  │
│                               Prepare Reference-to-Video Request                     │
│                                                   │                                  │
│                                                   ↓                                  │
│                               Submit Reference-to-Video                              │
│                                                   │                                  │
│                                                   ↓                                  │
│                               Check R2V Status ←──────────────────────┐              │
│                                                   │                   │              │
│                                                   ↓                   │              │
│                                            R2V Complete?              │              │
│                                          /    |    |    \             │              │
│                                   COMPLETED FAILED IN_PROGRESS IN_QUEUE             │
│                                        ↓      ↓         ↓        ↓    │              │
│                                   Get R2V   Log     Wait for R2V ─────┘              │
│                                   Result   Error    (20 seconds)                    │
│                                        │      │                                      │
│                                        ↓      ↓                                      │
│                              Archive Video + Notify User                             │
│                                        │                                             │
│                                        ↓                                             │
│                                  Route Output                                        │
│                                   /        \                                         │
│                            Telegram      Email                                       │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Cost Comparison

| Approach | API Calls | Est. Cost | Processing Time |
|----------|-----------|-----------|-----------------|
| **Kling O1 (Recommended)** | 1 | ~$0.25 | ~60-120 seconds |
| **WAN 2.6 Two-Phase** | 3 | ~$0.75 | ~180-300 seconds |

**WAN 2.6 Cost Breakdown:**
- Phase 1a: User I2V (5s) = ~$0.20
- Phase 1b: Picasso I2V (5s) = ~$0.20
- Phase 2: R2V (10s) = ~$0.35
- **Total:** ~$0.75 per video

---

## Limitations Summary

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| No direct image-to-multi-video | Requires two-phase workaround | Use Kling O1 instead |
| R2V requires video inputs | Cannot skip Phase 1 | Accept cost/time overhead |
| R2V max 10 seconds | Shorter final video | Prompt for more action per second |
| Identity may drift in Phase 1 | User/Picasso appearance could change | Use detailed prompts with identity cues |
| 3x API calls | Higher cost, more failure points | Add robust error handling |

---

## When to Use WAN 2.6

**Use WAN 2.6 only if:**
1. Kling O1 identity preservation is insufficient after testing
2. You need R2V-specific features (multi-shot with video references)
3. You already have reference videos (skipping Phase 1)

**Use Kling O1 if:**
1. You have IMAGE inputs (not videos) - **YOUR USE CASE**
2. You need direct multi-identity video generation
3. You want simpler workflow (1 API call vs 3)
4. Cost efficiency is important

---

## Error Handling

Same three-tier strategy as Kling:
1. **Retry on IN_PROGRESS** - Continue polling up to 30 attempts per phase
2. **Log ALL failures** - Google Sheets captures each phase's errors
3. **Notify user** - Only after retries exhausted

**Additional considerations for two-phase:**
- If Phase 1a fails, abort entire workflow
- If Phase 1b fails, abort entire workflow
- If Phase 2 fails, log which reference videos were created

---

## Sources

| Source | URL | Accessed |
|--------|-----|----------|
| fal.ai WAN 2.6 I2V API | https://fal.ai/models/wan/v2.6/image-to-video/api | 2026-01-06 |
| fal.ai WAN 2.6 R2V API | https://fal.ai/models/wan/v2.6/reference-to-video/api | 2026-01-06 |
| n8n HTTP Request Node | n8n-mcp (local) | 2026-01-06 |
| n8n Set Node | n8n-mcp (local) | 2026-01-06 |

---

## Related Documents

- `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` - **Recommended primary solution**
- `docs/analysis/analysis-tool-comparative-analysis-v1.0-2026-01-06.md` - Tool comparison
- `docs/analysis/analysis-workflow-reconfiguration-plan-v1.0-2026-01-06.md` - Implementation plan
