# Picture with Picasso - n8n Workflow Design Document

**Version**: 2.1
**Created**: 2025-11-26
**Updated**: 2025-11-27
**Project**: Picture with Picasso
**Purpose**: Comprehensive prompt and technical specifications for generating n8n workflow JSONs

> **IMPORTANT**: This workflow requires **3 separate n8n workflows** due to n8n's single-trigger-per-workflow limitation. See [Architecture Change](#architecture-change-3-workflows) section.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Change: 3 Workflows](#architecture-change-3-workflows)
3. [Research Findings](#research-findings)
4. [Platform Recommendations](#platform-recommendations)
5. [Workflow Architecture](#workflow-architecture)
6. [n8n Generation Prompt](#n8n-generation-prompt)
7. [API Configuration Details](#api-configuration-details)
8. [Environment Variables & Credentials](#environment-variables--credentials)
9. [Cost Estimates](#cost-estimates)

---

## Project Overview

### Concept (v2.0)

A user sends a photo of themselves via **Telegram bot** or **web upload form**. The workflow:

1. **Receives** the user's photo via Telegram OR web form (captures email address)
2. **Randomly selects** one of several pre-stored Picasso photos from Google Drive (cycling through available content)
3. **Generates a video** using Kie.ai Wan 2.5 AI that animates the user "horsing around" with Picasso
4. **Delivers** the video back via Telegram OR email (based on input source)

### Key Changes from v1.0

| Aspect | v1.0 | v2.0 |
|--------|------|------|
| **Video Engine** | Replicate Face Swap + Omni-Human | Kie.ai Wan 2.5 (allows famous figures) |
| **Output** | Static image OR video | Video only |
| **Input Methods** | Telegram only | Telegram OR Web Form |
| **Output Methods** | Telegram only | Telegram OR Email |
| **Email Capture** | Not required | Required for all inputs |

### Why Wan 2.5?

Wan 2.5 (Alibaba) is specifically chosen because it **allows the use of famous/historical figures without guardrails**, unlike:
- Google Veo/Gemini/Imagen (blocks "prominent people")
- Flux/Black Forest Labs (trained to avoid celebrities)
- OpenAI Sora (content restrictions)

### User Flows

**Flow 1: Telegram Bot**
```
User takes photo → Sends to Telegram bot → Receives video via Telegram
```

**Flow 2: Web Upload**
```
User visits web page → Uploads photo + enters email → Receives video via email
```

---

## Architecture Change: 3 Workflows

### Critical Discovery

**n8n limitation**: Each workflow can only have ONE trigger node. The original design assumed dual triggers (Form + Telegram) in a single workflow, but this is not possible.

### Solution: Split into 3 Workflows

```
WORKFLOW 1: "Picture with Picasso - Web Form"
┌─────────────────┐
│  Form Trigger   │ ──► Normalize Data ──► Execute Workflow (Core)
└─────────────────┘

WORKFLOW 2: "Picture with Picasso - Telegram"
┌─────────────────┐
│ Telegram Trigger│ ──► Extract Email ──► Validate ──► Execute Workflow (Core)
└─────────────────┘

WORKFLOW 3: "Picture with Picasso - Core Processing" (Sub-workflow)
┌─────────────────┐
│ Execute Workflow│ ──► Picasso Selection ──► API Video Gen ──► Delivery
│    Trigger      │
└─────────────────┘
```

### Files to Generate

| Workflow | Filename | Purpose |
|----------|----------|---------|
| 1 | `picture-with-picasso-web.json` | Web form input handler |
| 2 | `picture-with-picasso-telegram.json` | Telegram bot input handler |
| 3 | `picture-with-picasso-core.json` | Shared processing sub-workflow |

---

## Research Findings

### Kie.ai Wan 2.5 API Capabilities

| Feature | Specification |
|---------|---------------|
| **API Provider** | Kie.ai (aggregator for Alibaba Wan 2.5) |
| **Image-to-Video Endpoint** | `wan/2-5-image-to-video` (wan2.5-i2v-preview) |
| **Text-to-Video Endpoint** | `wan/2-5-text-to-video` (wan2.5-t2v-preview) |
| **Duration Options** | 5 seconds, 10 seconds |
| **Resolution Options** | 720p, 1080p |
| **Aspect Ratios (T2V)** | 16:9, 9:16, 1:1 |
| **Features** | Native audio sync, lip-sync, prompt expansion, negative prompts |
| **Pricing (720p)** | 12 credits/sec (~$0.06/sec) |
| **Pricing (1080p)** | 20 credits/sec (~$0.10/sec) |
| **Famous Figures** | ALLOWED - no content restrictions |

**API Documentation**: https://docs.kie.ai/

### Wan 2.5 vs Competitors

| Feature | Wan 2.5 (Kie.ai) | Veo 3 (Google) | Sora 2 (OpenAI) |
|---------|------------------|----------------|-----------------|
| Famous Figures | **ALLOWED** | BLOCKED | BLOCKED |
| Audio Sync | Native, built-in | Less integrated | Limited |
| I2V Support | Yes | Yes | Yes |
| Max Duration | 10 seconds | ~8 seconds | Variable |
| Pricing | $0.06-0.10/sec | Higher | Higher |

### n8n Node Capabilities

**n8n Form Trigger (v2.3)**
- File upload field type with accepted formats
- Email field with validation
- Required field enforcement
- Custom CSS styling
- Response modes: on submission, after workflow

**n8n Telegram Trigger (v1.2)**
- Triggers: message, photo, callback_query
- Auto-download photos option
- Filter by chat IDs, user IDs
- Returns binary data + metadata

**n8n Send Email (v2.1)**
- SMTP-based delivery
- Attachment support
- HTML content

---

## Platform Recommendations

### Recommended Technology Stack (v2.0)

| Component | Service | Model/API | Reason |
|-----------|---------|-----------|--------|
| **Web Input** | n8n Form Trigger | Native v2.3 | File upload + email capture |
| **Telegram Input** | n8n Telegram Trigger | Native v1.2 | Mobile-friendly bot interface |
| **Picasso Storage** | Google Drive | Native n8n | Random selection, cycling |
| **Video Generation** | Kie.ai | wan2.5-i2v-preview | Allows famous figures, audio sync |
| **Email Delivery** | n8n Send Email | SMTP | Native n8n integration |
| **Telegram Delivery** | n8n Telegram | Native v1.2 | Send video back to user |

### Services to AVOID

- **Google Veo, Gemini, Imagen** - blocks famous figures
- **Flux/Black Forest Labs** - trained to avoid celebrities
- **OpenAI Sora** - content restrictions on public figures
- **Runway Gen-3** - may have similar restrictions

---

## Workflow Architecture

### Visual Flow Diagram (v2.0)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SECTION 1: DUAL INPUT TRIGGERS                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌──────────────────┐              ┌──────────────────┐                        │
│   │  n8n Form Trigger │              │  Telegram Trigger │                       │
│   │  (Web Upload)    │              │  (Bot Messages)   │                       │
│   │                  │              │                   │                       │
│   │  Fields:         │              │  Extracts:        │                       │
│   │  • Photo (file)  │              │  • Photo binary   │                       │
│   │  • Email (req)   │              │  • Chat ID        │                       │
│   │  • Name (opt)    │              │  • Caption/email  │                       │
│   └────────┬─────────┘              └─────────┬─────────┘                       │
│            │                                  │                                  │
│            └──────────────┬───────────────────┘                                  │
│                           ▼                                                      │
│                   ┌──────────────────┐                                          │
│                   │   Merge + Set    │ (Normalize: source, email, photo)        │
│                   └────────┬─────────┘                                          │
└────────────────────────────┼────────────────────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    SECTION 2: PICASSO PHOTO SELECTION                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                   ┌──────────────────┐                                          │
│                   │  Google Drive    │ (List files in Picasso folder)           │
│                   │  List Files      │                                          │
│                   └────────┬─────────┘                                          │
│                            ▼                                                    │
│                   ┌──────────────────┐                                          │
│                   │   Code Node      │ (Random selection with cycling)          │
│                   │   JavaScript     │                                          │
│                   └────────┬─────────┘                                          │
│                            ▼                                                    │
│                   ┌──────────────────┐                                          │
│                   │  Google Drive    │ (Get shareable URL or download)          │
│                   │  Get File        │                                          │
│                   └────────┬─────────┘                                          │
└────────────────────────────┼────────────────────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    SECTION 3: KIE.AI WAN 2.5 VIDEO GENERATION                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                   ┌──────────────────┐                                          │
│                   │   HTTP Request   │ → Upload user photo to get URL           │
│                   │   (File Upload)  │   (Kie.ai File Upload API)               │
│                   └────────┬─────────┘                                          │
│                            ▼                                                    │
│                   ┌──────────────────┐                                          │
│                   │  HTTP Request    │ → Kie.ai Create I2V Task                 │
│                   │  POST wan/2-5-   │   (Picasso photo + user photo + prompt)  │
│                   │  image-to-video  │                                          │
│                   └────────┬─────────┘                                          │
│                            ▼                                                    │
│              ┌────▶┌──────────────────┐                                         │
│              │     │     Wait         │ (10-15 sec polling interval)            │
│              │     └────────┬─────────┘                                         │
│              │              ▼                                                   │
│              │     ┌──────────────────┐                                         │
│              │     │  HTTP Request    │ → Check Task Status                     │
│              │     │  GET /task/{id}  │                                         │
│              │     └────────┬─────────┘                                         │
│              │              ▼                                                   │
│              │     ┌──────────────────┐                                         │
│              └─NO──│    IF Node       │ (status === "completed"?)               │
│                    └────────┬─────────┘                                         │
│                             │ YES                                               │
│                             ▼                                                   │
│                   ┌──────────────────┐                                          │
│                   │     Set Node     │ (Extract video URL from response)        │
│                   └────────┬─────────┘                                          │
└────────────────────────────┼────────────────────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SECTION 4: DUAL OUTPUT DELIVERY                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                   ┌──────────────────┐                                          │
│                   │    IF Node       │ (source === "telegram" ?)                │
│                   └────────┬─────────┘                                          │
│                   ┌────────┴────────┐                                           │
│                   ▼                 ▼                                           │
│          ┌────────────────┐  ┌────────────────┐                                 │
│          │    Telegram    │  │   Send Email   │                                 │
│          │   Send Video   │  │  (SMTP)        │                                 │
│          │  to Chat ID    │  │  with video    │                                 │
│          └────────┬───────┘  └────────┬───────┘                                 │
│                   │                   │                                         │
│                   └─────────┬─────────┘                                         │
│                             ▼                                                   │
│                   ┌──────────────────┐                                          │
│                   │  Google Drive    │ (Archive result video)                   │
│                   │  Upload File     │                                          │
│                   └──────────────────┘                                          │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Required n8n Nodes

| Node Type | Purpose | Count |
|-----------|---------|-------|
| Form Trigger | Web upload input | 1 |
| Telegram Trigger | Bot input | 1 |
| Merge | Combine input branches | 1 |
| Set | Data normalization/extraction | 3-4 |
| Code (JavaScript) | Random selection, data processing | 2 |
| HTTP Request | Kie.ai API calls | 3-4 |
| Wait | Polling delays | 1-2 |
| IF | Conditional branching | 2-3 |
| Google Drive | Storage (list, download, upload) | 3 |
| Telegram | Send video delivery | 1 |
| Send Email | Email delivery | 1 |

---

## n8n Generation Prompt

```
# N8N WORKFLOW: "Picture with Picasso" v2.0

## WORKFLOW OVERVIEW
Create an n8n workflow that accepts user photos via Telegram bot OR web form, generates an AI video of the user "hanging out with Picasso" using Kie.ai Wan 2.5, and delivers the result via Telegram or email.

## TECHNICAL STACK

### Video Generation
- **Service**: Kie.ai API
- **Model**: wan2.5-i2v-preview (Image-to-Video)
- **Base URL**: https://api.kie.ai (confirm with docs)
- **Why**: Wan 2.5 allows famous historical figures without content restrictions

### Input Methods
1. **n8n Form Trigger** - Web upload page with:
   - Photo upload field (required)
   - Email field (required)
   - Name field (optional)

2. **Telegram Trigger** - Bot that accepts:
   - Photo messages
   - Caption should contain email (or prompt user)

### Storage
- **Google Drive** - Store Picasso photos, archive results

### Output Methods
1. **Telegram** - Send video back to chat
2. **Email (SMTP)** - Send video link to captured email

---

## SECTION 1: DUAL INPUT TRIGGERS

### Node: Form Trigger (Web Input)
- **Form Title**: "Picture with Picasso"
- **Fields**:
  - `photo` (File, required) - Accept: .jpg, .jpeg, .png, .webp
  - `email` (Email, required)
  - `name` (Text, optional)
- **Response**: "Thanks! Your video is being generated. Check your email in 2-3 minutes."

### Node: Telegram Trigger (Bot Input)
- **Trigger On**: message
- **Additional Fields**:
  - download: true (auto-download photos)
- **Extract**: photo binary, chat_id, message.caption (for email)

### Node: Set - Normalize Input (Web)
```javascript
// For Web Form workflow
return [{
  json: {
    source: 'web',
    email: $json['Your_Email'],
    userName: $json['Your_Name'] || 'Friend',
    chatId: null,
    timestamp: $now.toISO()
  }
}];
```

### Node: Code - Extract Email from Caption (Telegram)
```javascript
// For Telegram workflow - extract email from caption using regex
const caption = $json.message?.caption || '';
const emailRegex = /[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}/;
const emailMatch = caption.match(emailRegex);

return [{
  json: {
    source: 'telegram',
    email: emailMatch ? emailMatch[0] : 'MISSING',
    userName: $json.message?.from?.first_name || 'Friend',
    chatId: $json.message?.chat?.id,
    messageId: $json.message?.message_id,
    timestamp: $now.toISO()
  }
}];
```

### Node: IF - Check Email Exists (Telegram Only)
```
Condition: $json.email === 'MISSING'
True Branch: Send message asking for email
False Branch: Continue to Execute Workflow
```

### Node: Telegram - Request Email (If Missing)
```
Operation: sendMessage
Chat ID: {{ $json.chatId }}
Text: "Please send your photo again with your email address in the caption.\n\nExample: Send your photo with caption 'myemail@example.com'"
```

---

## SECTION 2: PICASSO PHOTO SELECTION

### Node: Google Drive - List Files
- **Operation**: Search Files/Folders
- **Query**: Files in "Picasso_Photos" folder
- **Filter**: mimeType contains 'image'

### Node: Code - Random Selection with Cycling
```javascript
const files = $input.first().json.files;
const totalFiles = files.length;

// Simple random selection (could add cycling logic with external state)
const randomIndex = Math.floor(Math.random() * totalFiles);
const selectedFile = files[randomIndex];

return [{
  json: {
    selectedFile,
    fileName: selectedFile.name,
    fileId: selectedFile.id,
    description: selectedFile.description || 'Picasso scene'
  }
}];
```

### Node: Google Drive - Get Shareable Link
- **Operation**: Share (or generate direct URL)
- **File ID**: `{{ $json.fileId }}`

---

## SECTION 3: KIE.AI WAN 2.5 VIDEO GENERATION

### Node: HTTP Request - Upload User Photo (if needed)
Some APIs require hosted URLs. Use Kie.ai File Upload API or upload to temporary hosting.

### Node: HTTP Request - Create Video Task
- **Method**: POST
- **URL**: Kie.ai endpoint for wan/2-5-image-to-video
- **Authentication**: Bearer [KIE_API_KEY]
- **Headers**: `{ "Content-Type": "application/json" }`
- **Body**:
```json
{
  "prompt": "The person in the image is having a fun, playful moment with Pablo Picasso. They are laughing, gesturing, and enjoying each other's company. Picasso is animated and expressive. Ambient audio: casual conversation, laughter, artistic studio atmosphere. Camera: medium shot, natural movement.",
  "image_url": "{{ $json.userPhotoUrl }}",
  "duration": "5",
  "resolution": "720p",
  "enable_prompt_expansion": true
}
```

### Node: Code - Initialize Retry Counter
```javascript
// Initialize retry counter before polling loop
return [{
  json: {
    ...$json,
    retryCount: 0,
    maxRetries: 20 // 20 * 15 sec = 5 minutes max
  }
}];
```

### Node: Wait
- **Duration**: 15 seconds (polling interval)
- **Mode**: timeInterval

### Node: HTTP Request - Check Task Status
- **Method**: GET
- **URL**: `https://pollo.ai/api/platform/generation/status/{{ $json.taskId }}`
- **Headers**: `x-api-key: [POLLO_API_KEY]`

### Node: Code - Increment Retry Counter
```javascript
// Track polling attempts
const currentRetry = $json.retryCount || 0;
const maxRetries = $json.maxRetries || 20;

if (currentRetry >= maxRetries) {
  throw new Error('Video generation timed out after ' + (maxRetries * 15) + ' seconds');
}

return [{
  json: {
    ...$json,
    retryCount: currentRetry + 1
  }
}];
```

### Node: IF - Check Completion
- **Condition**: `$json.status === "succeed"` OR `$json.status === "failed"`
- **True**: Continue to delivery or error handling
- **False**: Loop back to Wait node

### Node: IF - Check Success vs Failure
- **Condition**: `$json.status === "succeed"`
- **True**: Continue to delivery
- **False**: Send error notification

### Node: Telegram - Send Error (On Failure)
```
Operation: sendMessage
Chat ID: {{ $json.chatId }}
Text: "Sorry, we encountered an error generating your video. Please try again later."
```

### Node: Set - Extract Video URL
- **Video URL**: `{{ $json.output.videoUrl }}`

---

## SECTION 4: DUAL OUTPUT DELIVERY

### Node: IF - Determine Delivery Method
- **Condition**: `$json.source === "telegram"`

### TRUE BRANCH (Telegram):

#### Node: Telegram - Send Video
- **Operation**: Send Video
- **Chat ID**: `{{ $json.chatId }}`
- **Video**: `{{ $json.videoUrl }}`
- **Caption**: "Here you are hanging out with Pablo Picasso! 🎨🖼️"

### FALSE BRANCH (Email):

#### Node: HTTP Request - Download Video for Email
- **Method**: GET
- **URL**: `{{ $json.output.videoUrl }}`
- **Response Format**: file
- **Output Binary Property**: `videoData`

**Why download first?**: Video URL from API may expire. Downloading ensures we can attach to email.

#### Node: Send Email (SMTP)
- **From**: `{{ $env.EMAIL_FROM }}`
- **To**: `{{ $json.email }}`
- **Subject**: "Your Picture with Picasso is Ready! 🎨"
- **Email Format**: HTML
- **Attachments**: `videoData` (from previous download step)
- **HTML Body**:
```html
<div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
  <h1 style="color: #1a1a2e;">Your Picture with Picasso! 🎨</h1>
  <p>Hi {{ $json.userName }},</p>
  <p>Your AI-generated video with Pablo Picasso is ready!</p>
  <p>The video is attached to this email. You can also <a href="{{ $json.output.videoUrl }}">click here to view it online</a> (link expires in 3 days).</p>
  <p>Thanks for using Picture with Picasso!</p>
  <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;">
  <p style="color: #666; font-size: 12px;">This is an automated message.</p>
</div>
```

### Node: Google Drive - Archive Result
- **Operation**: Upload File
- **Folder**: Picture_With_Picasso_Results
- **File Name**: `{{ $json.timestamp }}_{{ $json.email }}.mp4`

---

## ERROR HANDLING

### Error Handling Nodes (Required)

| Error Type | Detection | Action |
|------------|-----------|--------|
| Missing email (Telegram) | `email === 'MISSING'` | Send Telegram message requesting email |
| API timeout | `retryCount >= maxRetries` | Throw error, notify user |
| Video generation failed | `status === 'failed'` | Send error notification via Telegram/Email |
| Invalid image format | File type check | Reject with message |
| Rate limit hit | 429 response | Wait and retry (exponential backoff) |

### Error Notification Logic

**For Telegram users**:
```
Telegram sendMessage to chatId:
"Sorry, we encountered an error generating your video. Please try again later."
```

**For Web users**:
```
Email to user:
Subject: "Issue with your Picture with Picasso request"
Body: "We encountered an error generating your video. Please try submitting again."
```

### Timeout Configuration

- **Polling interval**: 15 seconds
- **Max retries**: 20 (5 minutes total)
- **Video generation typically**: 60-120 seconds

---

## WORKFLOW JSON STRUCTURE REQUIREMENTS

Generate valid n8n workflow JSON with:
- Proper node positioning (x, y coordinates)
- Correct connection mappings between nodes
- All credential placeholders marked
- Sticky notes explaining each section
- Version compatibility: n8n 1.0+
```

---

## API Configuration Details

### API Provider Recommendation

| Provider | Status | Reason |
|----------|--------|--------|
| **Pollo AI** | **PRIMARY (Recommended)** | Clear documentation, well-defined endpoints |
| Kie.ai | SECONDARY | Base URL discovered, but Wan 2.5 endpoint not documented |

---

### Pollo AI (PRIMARY - Wan 2.5)

**Base URL**: `https://pollo.ai/api/platform`

**Authentication**:
```
x-api-key: [POLLO_API_KEY]
Content-Type: application/json
```

**Image-to-Video Endpoint**: `POST /generation/wanx/wan-v2-5-preview`

**Request Schema**:
```json
{
  "input": {
    "image": "string (required) - URL of source image",
    "prompt": "string (required) - Motion/animation description",
    "negativePrompt": "string (optional) - Content to avoid",
    "length": 5,
    "resolution": "720P | 1080P",
    "aspectRatio": "16:9 | 9:16 | 1:1",
    "wanAudio": true
  },
  "webhookUrl": "string (optional) - Callback URL"
}
```

**Response (Task Created)**:
```json
{
  "taskId": "string",
  "status": "waiting | processing"
}
```

**Status Check**: `GET /generation/status/{taskId}`

**Response (Task Complete)**:
```json
{
  "taskId": "string",
  "status": "succeed",
  "output": {
    "videoUrl": "string",
    "duration": "number",
    "resolution": "string"
  }
}
```

---

### Kie.ai API (SECONDARY - File Upload)

**Base URL**: `https://kieai.redpandaai.co` (NOT `api.kie.ai`)

**Authentication**:
```
Authorization: Bearer [KIE_API_KEY]
```

**File Upload Endpoints**:
- `POST /api/file-url-upload` - Upload from URL
- `POST /api/file-stream-upload` - Stream upload (max 10MB)
- `POST /api/file-base64-upload` - Base64 upload

**File Upload Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "fileId": "string",
    "fileName": "string",
    "fileUrl": "string (use this for Wan 2.5)",
    "downloadUrl": "string",
    "expiresAt": "ISO8601 (3 days from upload)"
  }
}
```

**IMPORTANT**: Files expire after 3 days. Download video before expiry for archival.

**Image-to-Video Model**: `wan2.5-i2v-preview` (endpoint not documented - use Pollo AI instead)

### Google Drive API (via n8n)

- **List Files**: Get all Picasso photos from designated folder
- **Get File**: Download or get shareable link
- **Upload File**: Archive generated videos

### Email (SMTP)

Configure n8n SMTP credentials:
- Host, Port, User, Password
- Or use services like SendGrid, Mailgun

---

## Environment Variables & Credentials

### Required Environment Variables

| Variable | Description | Example | Sensitive |
|----------|-------------|---------|-----------|
| `POLLO_API_KEY` | Pollo AI API key for Wan 2.5 | `pk_live_...` | Yes |
| `KIE_API_KEY` | Kie.ai API key for file uploads | `kie_...` | Yes |
| `PICASSO_FOLDER_ID` | Google Drive folder with Picasso photos | `1aHRwLWyrq...` | No |
| `ARCHIVE_FOLDER_ID` | Google Drive folder for results | `1bXsT9zk...` | No |
| `EMAIL_FROM` | Sender email for SMTP | `noreply@example.com` | No |
| `N8N_WEBHOOK_URL` | n8n base URL for callbacks | `https://n8n.domain.com/webhook` | No |

### Required Credentials

| Credential Name | Type | Description | Required Fields |
|-----------------|------|-------------|-----------------|
| `telegramApi` | Telegram Bot API | Bot token from @BotFather | accessToken |
| `googleDriveOAuth2Api` | Google OAuth2 | Drive access for photos/archive | clientId, clientSecret |
| `smtp` | SMTP | Email delivery | host, port, user, password, secure |

### Setup Checklist

- [ ] Create Pollo AI account and generate API key
- [ ] Create Kie.ai account and generate API key (for file uploads)
- [ ] Create Telegram bot via @BotFather, save token
- [ ] Create Google Cloud project with Drive API enabled
- [ ] Create OAuth2 credentials for Google Drive
- [ ] Create two Google Drive folders (Picasso photos + Archive)
- [ ] Configure SMTP credentials (or use SendGrid/Mailgun)

---

## Cost Estimates

### Kie.ai Wan 2.5 Pricing

| Resolution | Credits/Second | $/Second | 5 sec video | 10 sec video |
|------------|----------------|----------|-------------|--------------|
| 720p | 12 | ~$0.06 | ~$0.30 | ~$0.60 |
| 1080p | 20 | ~$0.10 | ~$0.50 | ~$1.00 |

### Per Execution Costs (v2.0)

| Component | Estimated Cost |
|-----------|----------------|
| Kie.ai Wan 2.5 (5 sec, 720p) | ~$0.30 |
| Google Drive API | Free |
| Email (SMTP) | Free/minimal |
| n8n (self-hosted) | Free |

### Monthly Cost Estimates

| Usage Level | Videos/Month | Cost (720p, 5 sec) |
|-------------|--------------|-------------------|
| Light | 100 | ~$30 |
| Medium | 500 | ~$150 |
| Heavy | 1,000 | ~$300 |

---

## Picasso Scene Photos to Prepare

Store these historical Picasso photos in Google Drive folder "Picasso_Photos":

1. **Beach Scene** - Picasso at the beach (Cote d'Azur, 1950s)
2. **Studio Scene** - Picasso in his studio (Mougins, 1960s)
3. **Cafe Scene** - Picasso at a Paris cafe (1920s)
4. **Gallery Scene** - Picasso at an art gallery opening
5. **Dinner Scene** - Picasso dining with friends (1940s)
6. **Garden Scene** - Picasso in a garden setting
7. **Portrait Scene** - Close-up candid of Picasso

**Image Requirements**:
- Clear view of Picasso
- Public domain (70+ years old)
- Good lighting and quality
- High resolution (minimum 1024x1024)
- Variety of settings for diversity

---

## Web Form Design

### Suggested Form Layout

```html
<!-- n8n Form Trigger will generate this, but here's the concept -->
<form>
  <h1>Picture with Picasso 🎨</h1>
  <p>Upload your photo and we'll create a fun video of you hanging out with Pablo Picasso!</p>

  <label>Your Photo *</label>
  <input type="file" accept=".jpg,.jpeg,.png,.webp" required />

  <label>Your Email *</label>
  <input type="email" required placeholder="you@example.com" />

  <label>Your Name (optional)</label>
  <input type="text" placeholder="Optional" />

  <button type="submit">Create My Video!</button>

  <p><small>Your video will be emailed to you in 2-3 minutes.</small></p>
</form>
```

### Custom CSS (for n8n Form Trigger)

```css
.form-container {
  max-width: 500px;
  margin: 0 auto;
  font-family: 'Segoe UI', sans-serif;
}
h1 { color: #1a1a2e; }
.submit-btn {
  background: #e94560;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
```

---

## Telegram Bot Setup

### Bot Commands

```
/start - Welcome message with instructions
/help - How to use the bot
/status - Check if workflow is active
```

### Welcome Message

```
Welcome to Picture with Picasso! 🎨

Send me a photo of yourself and I'll create a fun video of you hanging out with Pablo Picasso!

To get your video:
1. Send me a photo
2. Include your email in the caption (e.g., "myemail@example.com")
3. Wait 2-3 minutes for your video!

Example: Send a selfie with caption "john@email.com"
```

---

## Next Steps

1. [ ] Sign up for Kie.ai account and get API key
2. [ ] Gather public domain Picasso photographs (5-7 images)
3. [ ] Upload Picasso photos to Google Drive folder
4. [ ] Set up Telegram bot via BotFather
5. [ ] Configure SMTP credentials in n8n
6. [ ] Generate n8n workflow JSON from this prompt
7. [ ] Test both input methods (Telegram + Web Form)
8. [ ] Test both output methods (Telegram + Email)
9. [ ] Deploy and monitor

---

## Validation Reference

For detailed JSON schema sticky notes and one-shot generation strategy, see:
**`WORKFLOW-VALIDATION-REPORT.md`**

Contains:
- 15 JSON schema sticky notes for all nodes
- Missing component specifications
- Pre-generation checklist
- Test sequence recommendations

---

*Document updated by Claude Code analysis on 2025-11-27*
*Version 2.1 - Split into 3 workflows + Pollo AI primary + Validated Architecture*

### Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-26 | Initial design with Replicate Face Swap |
| 2.0 | 2025-11-27 | Switched to Kie.ai Wan 2.5, added dual input/output |
| 2.1 | 2025-11-27 | Split to 3 workflows, added Pollo AI, added retry logic, email validation |
