# Picture with Picasso - Workflow Validation Report

**Version**: 1.0
**Date**: 2025-11-27
**Purpose**: Comprehensive validation of n8n workflow architecture with JSON schema sticky notes for one-shot JSON generation

---

## Executive Summary

### Validation Status: NEEDS REFINEMENT

| Section | Status | Issues Found |
|---------|--------|--------------|
| Section 1: Dual Input | PARTIAL | Missing email extraction logic for Telegram |
| Section 2: Photo Selection | NEEDS UPDATE | Missing Google Drive folder ID, cycling logic |
| Section 3: Kie.ai Integration | CRITICAL GAP | Base URL incorrect, missing task status endpoint |
| Section 4: Dual Output | PARTIAL | Missing video download before email attachment |

---

## CRITICAL GAPS IDENTIFIED

### Gap 1: Kie.ai API Base URL Mismatch

**Issue**: Design document shows `https://api.kie.ai` but actual base URL is:
```
https://kieai.redpandaai.co
```

**Endpoints discovered**:
- File Upload: `POST /api/file-url-upload`
- File Stream: `POST /api/file-stream-upload`
- File Base64: `POST /api/file-base64-upload`

**MISSING**: Wan 2.5 video generation endpoint not documented in Kie.ai docs.

**ALTERNATIVE FOUND**: Pollo AI has clearer Wan 2.5 API:
```
POST https://pollo.ai/api/platform/generation/wanx/wan-v2-5-preview
```

### Gap 2: Missing Task Status Polling Endpoint

**Issue**: No documented endpoint to check video generation status on Kie.ai.

**Required**: Need task status endpoint like:
```
GET /api/task/{taskId}
```

### Gap 3: Telegram Email Extraction Missing

**Issue**: Current design assumes email in caption but no validation/extraction logic.

**Required**: Regex extraction + fallback prompt to user.

### Gap 4: File Lifecycle Management

**Issue**: Kie.ai files expire after 3 days. Design doesn't account for:
- Downloading video before expiry
- Permanent storage strategy

### Gap 5: Two Separate Workflows Needed

**Issue**: Cannot have TWO triggers (Form Trigger + Telegram Trigger) in the same workflow in n8n.

**Solution**: Need TWO workflows:
1. Web Form Workflow (Form Trigger)
2. Telegram Bot Workflow (Telegram Trigger)
Both call a shared sub-workflow for processing.

---

## SECTION-BY-SECTION VALIDATION

---

## SECTION 1: DUAL INPUT TRIGGERS

### Issue: n8n Limitation - One Trigger Per Workflow

n8n workflows can only have ONE trigger node. The dual-input architecture requires:

**Option A**: Two separate workflows sharing a sub-workflow
**Option B**: Webhook as unified entry point with routing logic

### Recommended Architecture:

```
WORKFLOW 1: "Picture with Picasso - Web Form"
┌─────────────────┐
│  Form Trigger   │ ──► Execute Workflow (Sub-workflow)
└─────────────────┘

WORKFLOW 2: "Picture with Picasso - Telegram"
┌─────────────────┐
│ Telegram Trigger│ ──► Execute Workflow (Sub-workflow)
└─────────────────┘

WORKFLOW 3: "Picture with Picasso - Core Processing" (Sub-workflow)
┌─────────────────┐
│ Execute Workflow│ ──► Picasso Selection ──► Kie.ai ──► Delivery
│    Trigger      │
└─────────────────┘
```

---

### STICKY NOTE 1: Form Trigger Configuration

```json
{
  "sticky_note_id": "SN_FORM_TRIGGER",
  "section": "Section 1 - Web Input",
  "node_type": "n8n-nodes-base.formTrigger",
  "version": "2.3",
  "configuration": {
    "formTitle": "Picture with Picasso",
    "formDescription": "Upload your photo and we'll create a fun video of you with Pablo Picasso!",
    "formFields": {
      "values": [
        {
          "fieldLabel": "Your Photo",
          "fieldType": "file",
          "requiredField": true,
          "acceptFileTypes": ".jpg, .jpeg, .png, .webp",
          "multipleFiles": false
        },
        {
          "fieldLabel": "Your Email",
          "fieldType": "email",
          "requiredField": true,
          "placeholder": "you@example.com"
        },
        {
          "fieldLabel": "Your Name",
          "fieldType": "text",
          "requiredField": false,
          "placeholder": "Optional"
        }
      ]
    },
    "responseMode": "onReceived",
    "options": {
      "respondWithOptions": {
        "values": {
          "respondWith": "text",
          "formSubmittedText": "Thanks! Your video is being generated. Check your email in 2-3 minutes."
        }
      },
      "appendAttribution": false
    }
  },
  "credentials_required": [],
  "output_schema": {
    "json": {
      "Your_Photo": "binary_reference",
      "Your_Email": "string",
      "Your_Name": "string | null",
      "submittedAt": "ISO8601_datetime"
    },
    "binary": {
      "Your_Photo": {
        "data": "base64_string",
        "mimeType": "image/jpeg | image/png | image/webp",
        "fileName": "string"
      }
    }
  }
}
```

---

### STICKY NOTE 2: Telegram Trigger Configuration

```json
{
  "sticky_note_id": "SN_TELEGRAM_TRIGGER",
  "section": "Section 1 - Telegram Input",
  "node_type": "n8n-nodes-base.telegramTrigger",
  "version": "1.2",
  "configuration": {
    "updates": ["message"],
    "additionalFields": {
      "download": true
    }
  },
  "credentials_required": [
    {
      "name": "telegramApi",
      "type": "telegramApi",
      "required_fields": ["accessToken"]
    }
  ],
  "output_schema": {
    "json": {
      "message": {
        "message_id": "number",
        "from": {
          "id": "number",
          "first_name": "string",
          "username": "string"
        },
        "chat": {
          "id": "number",
          "type": "private | group"
        },
        "date": "unix_timestamp",
        "photo": [
          {
            "file_id": "string",
            "file_unique_id": "string",
            "width": "number",
            "height": "number",
            "file_size": "number"
          }
        ],
        "caption": "string | undefined"
      }
    },
    "binary": {
      "data": {
        "data": "base64_string",
        "mimeType": "image/jpeg",
        "fileName": "string"
      }
    }
  },
  "notes": [
    "Email must be extracted from caption using regex",
    "If no email in caption, send message asking for email",
    "photo array contains multiple sizes - use largest (last item)"
  ]
}
```

---

### STICKY NOTE 3: Input Normalization (Set Node)

```json
{
  "sticky_note_id": "SN_NORMALIZE_INPUT",
  "section": "Section 1 - Data Normalization",
  "node_type": "n8n-nodes-base.set",
  "purpose": "Normalize input data from different sources into unified format",
  "configuration": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "name": "source",
          "value": "={{ $json.message ? 'telegram' : 'web' }}",
          "type": "string"
        },
        {
          "name": "email",
          "value": "={{ $json.message ? ($json.message.caption?.match(/[\\w.-]+@[\\w.-]+\\.[a-zA-Z]{2,}/)?.[0] || 'MISSING') : $json['Your_Email'] }}",
          "type": "string"
        },
        {
          "name": "chatId",
          "value": "={{ $json.message?.chat?.id || null }}",
          "type": "string"
        },
        {
          "name": "userName",
          "value": "={{ $json.message?.from?.first_name || $json['Your_Name'] || 'Friend' }}",
          "type": "string"
        },
        {
          "name": "timestamp",
          "value": "={{ $now.toISO() }}",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": true
  },
  "output_schema": {
    "source": "telegram | web",
    "email": "string",
    "chatId": "number | null",
    "userName": "string",
    "timestamp": "ISO8601_datetime"
  }
}
```

---

## SECTION 2: PICASSO PHOTO SELECTION

### STICKY NOTE 4: Google Drive List Files

```json
{
  "sticky_note_id": "SN_GDRIVE_LIST",
  "section": "Section 2 - Picasso Photo Selection",
  "node_type": "n8n-nodes-base.googleDrive",
  "version": "3",
  "configuration": {
    "operation": "list",
    "resource": "file",
    "folderId": {
      "__rl": true,
      "mode": "id",
      "value": "={{$env.PICASSO_FOLDER_ID}}"
    },
    "options": {
      "fields": ["id", "name", "webContentLink", "webViewLink", "mimeType"]
    }
  },
  "credentials_required": [
    {
      "name": "googleDriveOAuth2Api",
      "type": "googleDriveOAuth2Api",
      "required_fields": ["clientId", "clientSecret"]
    }
  ],
  "environment_variables": [
    {
      "name": "PICASSO_FOLDER_ID",
      "description": "Google Drive folder ID containing Picasso photos",
      "example": "1aHRwLWyrqfzoVC8HoB-YMrBvQ4tLC-NZ"
    }
  ],
  "output_schema": {
    "json": [
      {
        "id": "string",
        "name": "string",
        "mimeType": "image/jpeg | image/png",
        "webContentLink": "string (direct download URL)",
        "webViewLink": "string (view URL)"
      }
    ]
  }
}
```

---

### STICKY NOTE 5: Random Selection Code Node

```json
{
  "sticky_note_id": "SN_RANDOM_SELECT",
  "section": "Section 2 - Random Selection",
  "node_type": "n8n-nodes-base.code",
  "version": "2",
  "configuration": {
    "language": "javaScript",
    "mode": "runOnceForAllItems",
    "jsCode": "// Get all Picasso photos from previous node\nconst files = $input.all().map(item => item.json);\n\nif (files.length === 0) {\n  throw new Error('No Picasso photos found in Google Drive folder');\n}\n\n// Random selection\nconst randomIndex = Math.floor(Math.random() * files.length);\nconst selectedFile = files[randomIndex];\n\n// Get normalized input from earlier in workflow\nconst inputData = $('Normalize Input').first().json;\n\nreturn [{\n  json: {\n    ...inputData,\n    picasso: {\n      fileId: selectedFile.id,\n      fileName: selectedFile.name,\n      downloadUrl: selectedFile.webContentLink,\n      viewUrl: selectedFile.webViewLink\n    },\n    totalPicassoPhotos: files.length,\n    selectedIndex: randomIndex\n  }\n}];"
  },
  "output_schema": {
    "source": "string",
    "email": "string",
    "chatId": "number | null",
    "userName": "string",
    "timestamp": "string",
    "picasso": {
      "fileId": "string",
      "fileName": "string",
      "downloadUrl": "string",
      "viewUrl": "string"
    },
    "totalPicassoPhotos": "number",
    "selectedIndex": "number"
  }
}
```

---

## SECTION 3: KIE.AI WAN 2.5 INTEGRATION

### CRITICAL: API Provider Decision

**Option A: Kie.ai** (Original choice)
- Base URL: `https://kieai.redpandaai.co`
- Wan 2.5 endpoint: NOT DOCUMENTED in official docs
- File upload: Well documented
- Status: RISKY - may need to contact support for Wan API

**Option B: Pollo AI** (Alternative - CLEARER DOCS)
- Base URL: `https://pollo.ai/api/platform`
- Wan 2.5 endpoint: `/generation/wanx/wan-v2-5-preview`
- Well documented with clear schema
- Status: RECOMMENDED

### STICKY NOTE 6: Upload User Photo to Kie.ai

```json
{
  "sticky_note_id": "SN_UPLOAD_PHOTO",
  "section": "Section 3 - File Upload",
  "node_type": "n8n-nodes-base.httpRequest",
  "purpose": "Upload user photo to get hosted URL for Wan 2.5",
  "configuration": {
    "method": "POST",
    "url": "https://kieai.redpandaai.co/api/file-stream-upload",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "Bearer {{$env.KIE_API_KEY}}"
        }
      ]
    },
    "sendBody": true,
    "contentType": "multipart-form-data",
    "bodyParameters": {
      "parameters": [
        {
          "name": "file",
          "parameterType": "formBinaryData",
          "inputDataFieldName": "={{ $binary.data ? 'data' : 'Your_Photo' }}"
        },
        {
          "name": "uploadPath",
          "value": "picasso-uploads"
        }
      ]
    }
  },
  "environment_variables": [
    {
      "name": "KIE_API_KEY",
      "description": "Kie.ai API key from https://kie.ai/api-key",
      "sensitive": true
    }
  ],
  "expected_response": {
    "success": true,
    "code": 200,
    "data": {
      "fileId": "string",
      "fileName": "string",
      "fileUrl": "string (use this for Wan 2.5)",
      "downloadUrl": "string",
      "expiresAt": "ISO8601 (3 days)"
    }
  },
  "notes": [
    "Files expire after 3 days",
    "Max file size: 10MB for stream upload",
    "Supported formats: JPG, PNG, WebP"
  ]
}
```

---

### STICKY NOTE 7: Wan 2.5 Video Generation (Pollo AI Alternative)

```json
{
  "sticky_note_id": "SN_WAN25_GENERATE",
  "section": "Section 3 - Video Generation",
  "node_type": "n8n-nodes-base.httpRequest",
  "purpose": "Generate video using Wan 2.5 Image-to-Video",
  "api_provider": "Pollo AI (clearer documentation)",
  "configuration": {
    "method": "POST",
    "url": "https://pollo.ai/api/platform/generation/wanx/wan-v2-5-preview",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "x-api-key",
          "value": "={{$env.POLLO_API_KEY}}"
        },
        {
          "name": "Content-Type",
          "value": "application/json"
        }
      ]
    },
    "sendBody": true,
    "contentType": "json",
    "body": {
      "input": {
        "image": "={{$json.userPhotoUrl}}",
        "prompt": "The person in the image is having a fun, playful moment with Pablo Picasso. They are laughing together, making artistic gestures, and enjoying each other's company in an art studio. Picasso is animated and expressive, speaking enthusiastically about art. Ambient audio: casual conversation, artistic atmosphere, paint brushes. Camera: medium shot, gentle movement.",
        "negativePrompt": "blurry, distorted faces, extra limbs, bad anatomy, text, watermark",
        "length": 5,
        "resolution": "720P",
        "aspectRatio": "16:9",
        "wanAudio": true
      },
      "webhookUrl": "={{$env.N8N_WEBHOOK_URL}}/wan-callback"
    }
  },
  "environment_variables": [
    {
      "name": "POLLO_API_KEY",
      "description": "Pollo AI API key",
      "sensitive": true
    },
    {
      "name": "N8N_WEBHOOK_URL",
      "description": "n8n instance base URL for webhook callbacks",
      "example": "https://your-n8n.domain.com/webhook"
    }
  ],
  "expected_response": {
    "taskId": "string",
    "status": "waiting | processing | succeed | failed"
  },
  "pricing": {
    "720P_5sec": "$0.30 approx",
    "1080P_5sec": "$0.50 approx"
  }
}
```

---

### STICKY NOTE 8: Poll Task Status

```json
{
  "sticky_note_id": "SN_POLL_STATUS",
  "section": "Section 3 - Status Polling",
  "node_type": "n8n-nodes-base.httpRequest",
  "purpose": "Check video generation task status",
  "configuration": {
    "method": "GET",
    "url": "https://pollo.ai/api/platform/generation/status/={{$json.taskId}}",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "x-api-key",
          "value": "={{$env.POLLO_API_KEY}}"
        }
      ]
    }
  },
  "expected_response_success": {
    "taskId": "string",
    "status": "succeed",
    "output": {
      "videoUrl": "string (download this)",
      "duration": "number",
      "resolution": "string"
    }
  },
  "expected_response_processing": {
    "taskId": "string",
    "status": "processing",
    "progress": "number (0-100)"
  }
}
```

---

### STICKY NOTE 9: Wait Node for Polling

```json
{
  "sticky_note_id": "SN_WAIT_POLL",
  "section": "Section 3 - Polling Loop",
  "node_type": "n8n-nodes-base.wait",
  "version": "1.1",
  "configuration": {
    "resume": "timeInterval",
    "amount": 15,
    "unit": "seconds"
  },
  "notes": [
    "Wait 15 seconds between status checks",
    "Video generation typically takes 60-120 seconds",
    "Maximum 20 retries (5 minutes total)"
  ]
}
```

---

### STICKY NOTE 10: IF Node - Check Status

```json
{
  "sticky_note_id": "SN_IF_STATUS",
  "section": "Section 3 - Status Check",
  "node_type": "n8n-nodes-base.if",
  "version": "2.2",
  "configuration": {
    "conditions": {
      "options": {
        "version": 2,
        "caseSensitive": true,
        "typeValidation": "strict"
      },
      "combinator": "or",
      "conditions": [
        {
          "operator": {
            "type": "string",
            "operation": "equals"
          },
          "leftValue": "={{$json.status}}",
          "rightValue": "succeed"
        },
        {
          "operator": {
            "type": "string",
            "operation": "equals"
          },
          "leftValue": "={{$json.status}}",
          "rightValue": "failed"
        }
      ]
    }
  },
  "outputs": {
    "true": "Video ready or failed - proceed to delivery",
    "false": "Still processing - loop back to Wait"
  }
}
```

---

## SECTION 4: DUAL OUTPUT DELIVERY

### STICKY NOTE 11: IF Node - Delivery Method

```json
{
  "sticky_note_id": "SN_IF_DELIVERY",
  "section": "Section 4 - Delivery Routing",
  "node_type": "n8n-nodes-base.if",
  "version": "2.2",
  "configuration": {
    "conditions": {
      "options": {
        "version": 2,
        "caseSensitive": true,
        "typeValidation": "strict"
      },
      "combinator": "and",
      "conditions": [
        {
          "operator": {
            "type": "string",
            "operation": "equals"
          },
          "leftValue": "={{$json.source}}",
          "rightValue": "telegram"
        }
      ]
    }
  },
  "outputs": {
    "true": "Send via Telegram",
    "false": "Send via Email"
  }
}
```

---

### STICKY NOTE 12: Telegram Send Video

```json
{
  "sticky_note_id": "SN_TELEGRAM_SEND",
  "section": "Section 4 - Telegram Delivery",
  "node_type": "n8n-nodes-base.telegram",
  "version": "1.2",
  "configuration": {
    "resource": "message",
    "operation": "sendVideo",
    "chatId": "={{$json.chatId}}",
    "video": "={{$json.output.videoUrl}}",
    "additionalFields": {
      "caption": "Here you are hanging out with Pablo Picasso! 🎨🖼️\n\nGenerated by Picture with Picasso",
      "parseMode": "HTML"
    }
  },
  "credentials_required": [
    {
      "name": "telegramApi",
      "type": "telegramApi"
    }
  ],
  "notes": [
    "Video URL must be publicly accessible",
    "Max video size: 50MB for Telegram",
    "Supported formats: MP4"
  ]
}
```

---

### STICKY NOTE 13: Download Video for Email

```json
{
  "sticky_note_id": "SN_DOWNLOAD_VIDEO",
  "section": "Section 4 - Video Download",
  "node_type": "n8n-nodes-base.httpRequest",
  "purpose": "Download video file for email attachment",
  "configuration": {
    "method": "GET",
    "url": "={{$json.output.videoUrl}}",
    "responseFormat": "file",
    "options": {
      "response": {
        "response": {
          "fullResponse": false,
          "responseFormat": "file"
        }
      }
    }
  },
  "output_schema": {
    "binary": {
      "data": {
        "data": "base64_video_data",
        "mimeType": "video/mp4",
        "fileName": "picasso_video.mp4"
      }
    }
  }
}
```

---

### STICKY NOTE 14: Send Email with Video

```json
{
  "sticky_note_id": "SN_EMAIL_SEND",
  "section": "Section 4 - Email Delivery",
  "node_type": "n8n-nodes-base.emailSend",
  "version": "2.1",
  "configuration": {
    "resource": "email",
    "operation": "send",
    "fromEmail": "={{$env.EMAIL_FROM}}",
    "toEmail": "={{$json.email}}",
    "subject": "Your Picture with Picasso is Ready! 🎨",
    "emailFormat": "html",
    "html": "<div style=\"font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;\">\n  <h1 style=\"color: #1a1a2e;\">Your Picture with Picasso! 🎨</h1>\n  <p>Hi {{$json.userName}},</p>\n  <p>Your AI-generated video with Pablo Picasso is ready!</p>\n  <p>The video is attached to this email. You can also <a href=\"{{$json.output.videoUrl}}\">click here to view it online</a> (link expires in 3 days).</p>\n  <p>Thanks for using Picture with Picasso!</p>\n  <hr style=\"border: none; border-top: 1px solid #eee; margin: 20px 0;\">\n  <p style=\"color: #666; font-size: 12px;\">This is an automated message.</p>\n</div>",
    "options": {
      "attachments": "data",
      "appendAttribution": false
    }
  },
  "credentials_required": [
    {
      "name": "smtp",
      "type": "smtp",
      "required_fields": ["host", "port", "user", "password"]
    }
  ],
  "environment_variables": [
    {
      "name": "EMAIL_FROM",
      "description": "Sender email address",
      "example": "noreply@yourdomain.com"
    }
  ]
}
```

---

### STICKY NOTE 15: Archive to Google Drive

```json
{
  "sticky_note_id": "SN_ARCHIVE_VIDEO",
  "section": "Section 4 - Archival",
  "node_type": "n8n-nodes-base.googleDrive",
  "version": "3",
  "configuration": {
    "operation": "upload",
    "resource": "file",
    "name": "={{$json.timestamp}}_{{$json.email.replace('@', '_at_')}}.mp4",
    "folderId": {
      "__rl": true,
      "mode": "id",
      "value": "={{$env.ARCHIVE_FOLDER_ID}}"
    },
    "options": {}
  },
  "environment_variables": [
    {
      "name": "ARCHIVE_FOLDER_ID",
      "description": "Google Drive folder ID for archiving results"
    }
  ]
}
```

---

## MISSING COMPONENTS IDENTIFIED

### 1. Email Validation for Telegram Input

```json
{
  "sticky_note_id": "SN_EMAIL_VALIDATION",
  "section": "Missing - Email Validation",
  "node_type": "n8n-nodes-base.if",
  "purpose": "Check if email was extracted from Telegram caption",
  "configuration": {
    "conditions": {
      "combinator": "and",
      "conditions": [
        {
          "operator": {
            "type": "string",
            "operation": "equals"
          },
          "leftValue": "={{$json.email}}",
          "rightValue": "MISSING"
        }
      ]
    }
  },
  "on_true_action": "Send Telegram message asking for email",
  "on_false_action": "Continue with workflow"
}
```

### 2. Error Notification Node

```json
{
  "sticky_note_id": "SN_ERROR_NOTIFY",
  "section": "Missing - Error Handling",
  "node_type": "n8n-nodes-base.telegram",
  "purpose": "Notify user of video generation failure",
  "configuration": {
    "resource": "message",
    "operation": "sendMessage",
    "chatId": "={{$json.chatId}}",
    "text": "Sorry, we encountered an error generating your video. Please try again later."
  }
}
```

### 3. Retry Counter

```json
{
  "sticky_note_id": "SN_RETRY_COUNTER",
  "section": "Missing - Retry Logic",
  "node_type": "n8n-nodes-base.code",
  "purpose": "Track polling retries to prevent infinite loops",
  "configuration": {
    "jsCode": "const currentRetry = $json.retryCount || 0;\nconst maxRetries = 20;\n\nif (currentRetry >= maxRetries) {\n  throw new Error('Video generation timed out after ' + (maxRetries * 15) + ' seconds');\n}\n\nreturn [{\n  json: {\n    ...$json,\n    retryCount: currentRetry + 1\n  }\n}];"
  }
}
```

---

## ENVIRONMENT VARIABLES REQUIRED

```json
{
  "sticky_note_id": "SN_ENV_VARS",
  "section": "Configuration - Environment Variables",
  "variables": [
    {
      "name": "KIE_API_KEY",
      "description": "Kie.ai API key for file uploads",
      "required": true,
      "sensitive": true
    },
    {
      "name": "POLLO_API_KEY",
      "description": "Pollo AI API key for Wan 2.5 video generation",
      "required": true,
      "sensitive": true
    },
    {
      "name": "PICASSO_FOLDER_ID",
      "description": "Google Drive folder ID containing Picasso photos",
      "required": true,
      "example": "1aHRwLWyrqfzoVC8HoB-YMrBvQ4tLC-NZ"
    },
    {
      "name": "ARCHIVE_FOLDER_ID",
      "description": "Google Drive folder ID for archiving results",
      "required": true
    },
    {
      "name": "EMAIL_FROM",
      "description": "Sender email address for SMTP",
      "required": true,
      "example": "noreply@yourdomain.com"
    },
    {
      "name": "N8N_WEBHOOK_URL",
      "description": "Base URL of n8n instance for callbacks",
      "required": false,
      "example": "https://n8n.yourdomain.com/webhook"
    }
  ]
}
```

---

## CREDENTIALS REQUIRED

```json
{
  "sticky_note_id": "SN_CREDENTIALS",
  "section": "Configuration - Credentials",
  "credentials": [
    {
      "name": "telegramApi",
      "type": "telegramApi",
      "description": "Telegram Bot API token from @BotFather",
      "fields": ["accessToken"]
    },
    {
      "name": "googleDriveOAuth2Api",
      "type": "googleDriveOAuth2Api",
      "description": "Google OAuth2 for Drive access",
      "fields": ["clientId", "clientSecret"],
      "scopes": ["https://www.googleapis.com/auth/drive"]
    },
    {
      "name": "smtp",
      "type": "smtp",
      "description": "SMTP server credentials for sending email",
      "fields": ["host", "port", "user", "password", "secure"]
    }
  ]
}
```

---

## ONE-SHOT VALIDATION STRATEGY

### Pre-Generation Checklist

1. **API Access Verification**
   - [ ] Kie.ai account created and API key obtained
   - [ ] Pollo AI account created and API key obtained (if using)
   - [ ] Test file upload endpoint works
   - [ ] Test Wan 2.5 endpoint works with sample image

2. **Google Drive Setup**
   - [ ] Picasso photos folder created
   - [ ] At least 5 Picasso photos uploaded
   - [ ] Archive folder created
   - [ ] OAuth2 credentials configured

3. **Telegram Bot Setup**
   - [ ] Bot created via @BotFather
   - [ ] Bot token obtained
   - [ ] Test message sending works

4. **Email Setup**
   - [ ] SMTP credentials configured
   - [ ] Test email sending works

### JSON Generation Strategy

**Step 1**: Generate THREE separate workflows:
1. `picture-with-picasso-web.json` - Form Trigger workflow
2. `picture-with-picasso-telegram.json` - Telegram Trigger workflow
3. `picture-with-picasso-core.json` - Core processing sub-workflow

**Step 2**: Use this prompt structure for generation:
```
Generate n8n workflow JSON for Picture with Picasso [WEB/TELEGRAM/CORE] workflow.

Use EXACTLY these node configurations from the sticky notes:
[Paste relevant sticky note JSON schemas]

Connect nodes in this order:
[List node connection sequence]

Include these credentials placeholders:
[List credential types]

Include these environment variable references:
[List env vars]
```

**Step 3**: Validate each generated JSON:
- Import into n8n
- Check all nodes are connected
- Verify credential placeholders exist
- Test with sample data

### Recommended Test Sequence

1. Test Core Workflow first (with manual trigger)
2. Test Web Form Workflow
3. Test Telegram Workflow
4. End-to-end test both input paths

---

## SUMMARY OF CHANGES NEEDED TO DESIGN DOCUMENT

1. **Split into 3 workflows** (not 1)
2. **Update Kie.ai base URL** to `https://kieai.redpandaai.co`
3. **Add Pollo AI as primary API** for Wan 2.5 (clearer docs)
4. **Add email extraction regex** for Telegram input
5. **Add retry counter** for polling loop
6. **Add video download step** before email attachment
7. **Add error handling nodes**
8. **Add all environment variables**
9. **Add all credentials**

---

*Validation Report Generated: 2025-11-27*
*Ready for JSON Generation with Sticky Notes*
