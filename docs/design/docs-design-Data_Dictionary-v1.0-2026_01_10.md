# Picture with Picasso - Data Dictionary

**Version:** v1.0-2026-01-10
**Document Type:** Technical Reference
**Purpose:** Comprehensive data element catalog for workflow analytics, Supabase schema design, and metrics tracking
**Related:** Project Roadmap, Audio Pipeline v3.1, Compression Pipeline v3.2

---

## Document Overview

This data dictionary catalogs every data element flowing through the Picture with Picasso workflow, organized for:
1. **Supabase Schema Design** - Database table structure
2. **Event Logging** - Audit trail and debugging
3. **Cost Tracking** - Token/API consumption metrics
4. **Analytics** - Usage patterns and optimization

---

## Table of Contents

1. [User & Session Data](#1-user--session-data)
2. [Workflow Configuration](#2-workflow-configuration)
3. [Input Processing](#3-input-processing)
4. [API Interactions](#4-api-interactions)
5. [Cost & Token Tracking](#5-cost--token-tracking)
6. [Output & Delivery](#6-output--delivery)
7. [Supabase Schema Proposal](#7-supabase-schema-proposal)
8. [Metrics Tracking Summary](#8-metrics-tracking-summary)

---

## 1. User & Session Data

### 1.1 Session Identifiers

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `executionId` | string | n8n | Unique workflow execution ID | 1 |
| `workflowId` | string | n8n | Workflow identifier | 1 |
| `timestamp` | datetime | n8n | Execution start time | 1 |
| `inputSource` | enum | Set Node | `telegram` or `form` | 1 |

### 1.2 User Identifiers

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `chatId` | string | Telegram | Telegram chat ID for user | 1 |
| `telegramUserId` | string | Telegram | Telegram user ID | 1 |
| `telegramUsername` | string | Telegram | Telegram @username (if available) | 1 |
| `userEmail` | string | Form | User email address | 1 |
| `userId` | uuid | Supabase | Future: Authenticated user ID | 7 |

### 1.3 User Input Data

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `telegramFileId` | string | Telegram | File ID for uploaded photo | 1 |
| `uploadedImageUrl` | string | Form | URL of uploaded image | 1 |
| `userImageBase64` | string | Code Node | Base64-encoded user image | 1 |
| `originalImageSize` | number | Calculated | Original image size in bytes | 3 |
| `resizedImageSize` | number | Calculated | After resize to 2048px | 3 |

---

## 2. Workflow Configuration

### 2.1 Video Generation Settings

| Field | Type | Default | Description | Phase |
|-------|------|---------|-------------|-------|
| `duration` | string | `"5"` or `"10"` | Video duration in seconds | 1 |
| `resolution` | string | `"720p"` | Target resolution | 1 |
| `aspect_ratio` | string | `"9:16"` | Video aspect ratio | 1 |
| `optimizePrompt` | boolean | `true` | Enable prompt optimization | 1 |
| `generate_audio` | boolean | `true` | Request audio from Kling | 1 |

### 2.2 API Keys & Credentials (Reference Only - Not Logged)

| Field | Type | Service | Phase |
|-------|------|---------|-------|
| `falApiKey` | string | fal.ai (Kling, ElevenLabs, LatentSync) | 1 |
| `telegramBotToken` | credential | Telegram Bot API | 1 |
| `googleServiceAccount` | credential | Google Drive, Gmail | 1 |
| `cloudinaryCloudName` | string | Cloudinary | 3 |
| `cloudinaryApiKey` | string | Cloudinary | 3 |
| `cloudinaryApiSecret` | string | Cloudinary | 3 |

### 2.3 Folder IDs

| Field | Type | Purpose | Phase |
|-------|------|---------|-------|
| `videoArchiveFolderId` | string | Google Drive folder for generated videos | 1 |
| `userUploadedImagesFolderId` | string | Google Drive folder for user images | 1 |
| `composedImagesFolderId` | string | Google Drive folder for composed images | 1 |
| `picassoImagesFolderId` | string | Google Drive folder with Picasso references | 1 |

---

## 3. Input Processing

### 3.1 Picasso Reference Selection

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `selectedPicassoImageId` | string | Google Drive | ID of selected Picasso image | 1 |
| `selectedPicassoImageName` | string | Google Drive | Filename of Picasso image | 1 |
| `picassoImageBase64` | string | Code Node | Base64-encoded Picasso image | 1 |
| `picassoShuffleBagIndex` | number | Static Data | Position in shuffle bag | 1 |

### 3.2 Image Processing Metrics

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `userImageOriginalWidth` | number | Image | Original width in pixels | 3 |
| `userImageOriginalHeight` | number | Image | Original height in pixels | 3 |
| `userImageResizedWidth` | number | Image | After resize (max 2048) | 3 |
| `userImageResizedHeight` | number | Image | After resize (max 2048) | 3 |
| `imageResizeApplied` | boolean | Calculated | Whether resize was needed | 3 |

---

## 4. API Interactions

### 4.1 Kling O1 Video Generation

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `klingRequestId` | string | Submit Response | Unique Kling request ID | 1 |
| `klingStatusUrl` | string | Submit Response | URL to check status | 1 |
| `klingResponseUrl` | string | Submit Response | URL to get result | 1 |
| `klingStatus` | enum | Status Check | `IN_QUEUE`, `IN_PROGRESS`, `COMPLETED`, `FAILED` | 1 |
| `klingRetryCount` | number | Counter | Number of status polls | 1 |
| `klingMaxRetries` | number | Config | Maximum retries (30) | 1 |
| `klingWaitSeconds` | number | Config | Wait between polls (20s) | 1 |
| `klingVideoUrl` | string | Result | Generated video URL | 1 |
| `klingErrorMessage` | string | Result | Error details if failed | 1 |
| `klingProcessingTime` | number | Calculated | Total time in seconds | 1 |

#### 4.1.1 Kling Request Payload

| Field | Type | Description | Phase |
|-------|------|-------------|-------|
| `prompt` | string | Full video generation prompt | 1 |
| `negative_prompt` | string | Things to avoid | 1 |
| `elements` | array | Reference image elements | 1 |
| `elements[].reference_image_urls` | array | Base64 image array | 1 |
| `elements[].frontal_image_url` | string | Primary reference | 1 |

### 4.2 ElevenLabs TTS (Phase 2)

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `ttsRequestId` | string | Submit Response | ElevenLabs request ID | 2 |
| `ttsResponseUrl` | string | Submit Response | URL to get result | 2 |
| `ttsStatus` | enum | Status Check | `IN_QUEUE`, `IN_PROGRESS`, `COMPLETED`, `FAILED` | 2 |
| `ttsRetryCount` | number | Counter | Number of status polls | 2 |
| `ttsAudioUrl` | string | Result | Generated audio URL | 2 |
| `ttsProcessingTime` | number | Calculated | Total time in seconds | 2 |
| `dialogScript` | string | Config | Text to speak | 2 |
| `voiceId` | string | Config | ElevenLabs voice ID | 2 |
| `ttsCharacterCount` | number | Calculated | Characters in script | 2 |

### 4.3 LatentSync Lip Sync (Phase 2)

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `latentSyncRequestId` | string | Submit Response | LatentSync request ID | 2 |
| `latentSyncResponseUrl` | string | Submit Response | URL to get result | 2 |
| `latentSyncStatus` | enum | Status Check | `IN_QUEUE`, `IN_PROGRESS`, `COMPLETED`, `FAILED` | 2 |
| `latentSyncRetryCount` | number | Counter | Number of status polls | 2 |
| `latentSyncVideoUrl` | string | Result | Lip-synced video URL | 2 |
| `latentSyncProcessingTime` | number | Calculated | Total time in seconds | 2 |

### 4.4 Cloudinary Compression (Phase 3)

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `cloudinaryPublicId` | string | Upload Response | Cloudinary asset ID | 3 |
| `cloudinarySecureUrl` | string | Upload Response | HTTPS URL to asset | 3 |
| `cloudinaryTransformation` | string | Config | Applied transformations | 3 |
| `cloudinaryFormat` | string | Result | Output format (mp4, webm) | 3 |
| `cloudinaryOriginalSize` | number | Calculated | Size before compression | 3 |
| `cloudinaryCompressedSize` | number | Result | Size after compression | 3 |
| `cloudinaryCompressionRatio` | number | Calculated | Original/Compressed | 3 |
| `cloudinaryProcessingTime` | number | Calculated | Transform time in ms | 3 |

---

## 5. Cost & Token Tracking

### 5.1 API Cost Per Execution

| Service | Field | Type | Unit Cost | Calculation | Phase |
|---------|-------|------|-----------|-------------|-------|
| **Kling O1** | `klingCost` | number | $1.00/video | Fixed per 10s video | 1 |
| **ElevenLabs TTS** | `ttsCost` | number | ~$0.05/video | Based on character count | 2 |
| **LatentSync** | `latentSyncCost` | number | $0.20/video | Fixed per video | 2 |
| **Cloudinary** | `cloudinaryCost` | number | ~$0.01/video | Based on transformation | 3 |
| **Total** | `totalCost` | number | ~$1.26/video | Sum of all services | 4 |

### 5.2 Token/Credit Consumption

| Field | Type | Service | Description | Phase |
|-------|------|---------|-------------|-------|
| `falCreditsUsed` | number | fal.ai | Total fal.ai credits consumed | 1 |
| `klingCreditsUsed` | number | Kling O1 | Video generation credits | 1 |
| `elevenLabsCharacters` | number | ElevenLabs | Characters processed | 2 |
| `cloudinaryCreditsUsed` | number | Cloudinary | Transformation credits | 3 |

### 5.3 Processing Time Metrics

| Field | Type | Description | Phase |
|-------|------|-------------|-------|
| `totalProcessingTime` | number | End-to-end time in seconds | 1 |
| `imageProcessingTime` | number | Time for image resize/convert | 1 |
| `klingQueueTime` | number | Time in Kling queue | 1 |
| `klingGenerationTime` | number | Kling video generation | 1 |
| `ttsProcessingTime` | number | ElevenLabs TTS time | 2 |
| `lipSyncProcessingTime` | number | LatentSync time | 2 |
| `compressionTime` | number | Cloudinary transform time | 3 |
| `deliveryTime` | number | Time to send to user | 1 |

---

## 6. Output & Delivery

### 6.1 Generated Assets

| Field | Type | Source | Description | Phase |
|-------|------|--------|-------------|-------|
| `silentVideoUrl` | string | Kling | Original silent video URL | 1 |
| `audioUrl` | string | ElevenLabs | Generated TTS audio URL | 2 |
| `lipSyncedVideoUrl` | string | LatentSync | Video with lip sync | 2 |
| `compressedVideoUrl` | string | Cloudinary | Optimized video URL | 3 |
| `finalVideoUrl` | string | Cloudinary | Final delivery URL | 3 |

### 6.2 File Information

| Field | Type | Description | Phase |
|-------|------|-------------|-------|
| `silentVideoSizeBytes` | number | Kling output size | 1 |
| `audioSizeBytes` | number | TTS audio size | 2 |
| `lipSyncedVideoSizeBytes` | number | After lip sync | 2 |
| `compressedVideoSizeBytes` | number | After Cloudinary | 3 |
| `finalVideoSizeBytes` | number | Delivered file size | 3 |
| `compressionSavingsBytes` | number | Bytes saved | 3 |
| `compressionSavingsPercent` | number | Percentage reduction | 3 |

### 6.3 Filename Convention

| Field | Type | Example | Phase |
|-------|------|---------|-------|
| `internalFilename` | string | `PwP_telegram_2026-01-10_14-32-05_abc123.mp4` | 3 |
| `externalFilename` | string | `Picture_with_Picasso-AtelierBenito-abc123-2026_01_10.mp4` | 3 |

### 6.4 Delivery Tracking

| Field | Type | Description | Phase |
|-------|------|-------------|-------|
| `deliveryChannel` | enum | `telegram`, `email`, `web` | 1 |
| `deliveryStatus` | enum | `sent`, `failed`, `pending` | 1 |
| `deliveryTimestamp` | datetime | When delivered | 1 |
| `telegramMessageId` | string | Sent message ID | 1 |
| `emailMessageId` | string | Gmail message ID | 1 |
| `googleDriveFileId` | string | Archived file ID | 1 |
| `deliveryErrorMessage` | string | Error if failed | 1 |

### 6.5 Enhanced Delivery (Phase 4)

| Field | Type | Description | Phase |
|-------|------|-------------|-------|
| `messageTemplate` | string | Branded message text | 4 |
| `hashtags` | string | Social media hashtags | 4 |
| `attribution` | string | "An Atelier Benito joint..." | 4 |

---

## 7. Supabase Schema Proposal

### 7.1 Proposed Tables

#### Table: `executions`

Primary record for each workflow execution.

```sql
CREATE TABLE executions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  execution_id VARCHAR(50) UNIQUE NOT NULL,
  workflow_id VARCHAR(50) NOT NULL,

  -- User identification
  input_source VARCHAR(20) NOT NULL, -- 'telegram' or 'form'
  telegram_chat_id VARCHAR(50),
  telegram_user_id VARCHAR(50),
  telegram_username VARCHAR(100),
  user_email VARCHAR(255),
  user_id UUID REFERENCES users(id), -- Future: authenticated users

  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  completed_at TIMESTAMPTZ,

  -- Status
  status VARCHAR(20) DEFAULT 'pending', -- pending, processing, completed, failed
  error_message TEXT,

  -- Picasso selection
  picasso_image_id VARCHAR(100),
  picasso_image_name VARCHAR(255),

  -- Final output
  final_video_url TEXT,
  internal_filename VARCHAR(255),
  external_filename VARCHAR(255),
  google_drive_file_id VARCHAR(100),

  -- Delivery
  delivery_channel VARCHAR(20),
  delivery_status VARCHAR(20),
  delivery_timestamp TIMESTAMPTZ,
  telegram_message_id VARCHAR(50),
  email_message_id VARCHAR(100)
);

CREATE INDEX idx_executions_user ON executions(telegram_chat_id, user_email);
CREATE INDEX idx_executions_created ON executions(created_at);
CREATE INDEX idx_executions_status ON executions(status);
```

#### Table: `api_calls`

Detailed tracking of each external API interaction.

```sql
CREATE TABLE api_calls (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  execution_id UUID REFERENCES executions(id),

  -- API identification
  service VARCHAR(50) NOT NULL, -- 'kling', 'elevenlabs', 'latentsync', 'cloudinary'
  operation VARCHAR(50) NOT NULL, -- 'submit', 'status_check', 'get_result'

  -- Request tracking
  request_id VARCHAR(100),
  request_timestamp TIMESTAMPTZ DEFAULT NOW(),

  -- Response tracking
  response_status VARCHAR(50),
  response_timestamp TIMESTAMPTZ,

  -- Retry tracking
  retry_count INT DEFAULT 0,

  -- Cost tracking
  cost_usd DECIMAL(10,4),
  credits_used DECIMAL(10,4),

  -- Error tracking
  error_code VARCHAR(50),
  error_message TEXT
);

CREATE INDEX idx_api_calls_execution ON api_calls(execution_id);
CREATE INDEX idx_api_calls_service ON api_calls(service);
```

#### Table: `processing_metrics`

Performance and size metrics per execution.

```sql
CREATE TABLE processing_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  execution_id UUID REFERENCES executions(id) UNIQUE,

  -- Processing times (in milliseconds)
  total_processing_ms INT,
  image_processing_ms INT,
  kling_queue_ms INT,
  kling_generation_ms INT,
  tts_processing_ms INT,
  lip_sync_processing_ms INT,
  compression_ms INT,
  delivery_ms INT,

  -- File sizes (in bytes)
  user_image_original_bytes INT,
  user_image_resized_bytes INT,
  silent_video_bytes INT,
  audio_bytes INT,
  lip_synced_video_bytes INT,
  compressed_video_bytes INT,
  final_video_bytes INT,

  -- Compression metrics
  compression_ratio DECIMAL(5,2),
  compression_savings_percent DECIMAL(5,2),

  -- Retry counts
  kling_retries INT DEFAULT 0,
  tts_retries INT DEFAULT 0,
  lip_sync_retries INT DEFAULT 0
);

CREATE INDEX idx_metrics_execution ON processing_metrics(execution_id);
```

#### Table: `cost_tracking`

Aggregated cost data per execution.

```sql
CREATE TABLE cost_tracking (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  execution_id UUID REFERENCES executions(id) UNIQUE,

  -- Per-service costs (USD)
  kling_cost DECIMAL(10,4) DEFAULT 0,
  elevenlabs_cost DECIMAL(10,4) DEFAULT 0,
  latentsync_cost DECIMAL(10,4) DEFAULT 0,
  cloudinary_cost DECIMAL(10,4) DEFAULT 0,

  -- Total
  total_cost DECIMAL(10,4) GENERATED ALWAYS AS (
    kling_cost + elevenlabs_cost + latentsync_cost + cloudinary_cost
  ) STORED,

  -- Token/credit consumption
  fal_credits_used DECIMAL(10,4),
  elevenlabs_characters INT,
  cloudinary_credits DECIMAL(10,4),

  -- Metadata
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_cost_execution ON cost_tracking(execution_id);
CREATE INDEX idx_cost_recorded ON cost_tracking(recorded_at);
```

#### Table: `daily_aggregates`

Pre-computed daily summaries for dashboards.

```sql
CREATE TABLE daily_aggregates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date DATE UNIQUE NOT NULL,

  -- Volume metrics
  total_executions INT DEFAULT 0,
  successful_executions INT DEFAULT 0,
  failed_executions INT DEFAULT 0,

  -- Channel breakdown
  telegram_executions INT DEFAULT 0,
  form_executions INT DEFAULT 0,

  -- Cost totals
  total_cost_usd DECIMAL(10,2) DEFAULT 0,
  kling_cost_usd DECIMAL(10,2) DEFAULT 0,
  elevenlabs_cost_usd DECIMAL(10,2) DEFAULT 0,
  latentsync_cost_usd DECIMAL(10,2) DEFAULT 0,
  cloudinary_cost_usd DECIMAL(10,2) DEFAULT 0,

  -- Performance averages
  avg_processing_time_ms INT,
  avg_video_size_bytes INT,
  avg_compression_ratio DECIMAL(5,2),

  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_daily_date ON daily_aggregates(date);
```

---

## 8. Metrics Tracking Summary

### 8.1 Key Performance Indicators (KPIs)

| KPI | Formula | Target | Phase |
|-----|---------|--------|-------|
| **Success Rate** | Successful / Total | > 95% | 1 |
| **Avg Processing Time** | Sum(time) / Count | < 5 min | 1 |
| **Avg File Size** | Sum(bytes) / Count | 5-10 MB | 3 |
| **Compression Ratio** | Original / Compressed | 2-3x | 3 |
| **Cost Per Video** | Sum(costs) / Count | ~$1.26 | 4 |
| **Daily Volume** | Count per day | Track growth | 1 |

### 8.2 Excel Tracking Columns

For manual/spreadsheet tracking before Supabase implementation:

| Column | Type | Description |
|--------|------|-------------|
| A: Date | Date | Execution date |
| B: Time | Time | Execution time |
| C: Execution ID | Text | n8n execution ID |
| D: Input Source | Text | telegram/form |
| E: User ID | Text | Chat ID or email |
| F: Status | Text | completed/failed |
| G: Picasso Image | Text | Selected image name |
| H: Kling Time (s) | Number | Kling processing seconds |
| I: TTS Time (s) | Number | ElevenLabs seconds |
| J: Lip Sync Time (s) | Number | LatentSync seconds |
| K: Compress Time (s) | Number | Cloudinary seconds |
| L: Total Time (s) | Number | End-to-end seconds |
| M: Original Size (MB) | Number | Kling output size |
| N: Final Size (MB) | Number | After compression |
| O: Compression % | Number | Size reduction |
| P: Kling Cost | Currency | $1.00 |
| Q: TTS Cost | Currency | ~$0.05 |
| R: Lip Sync Cost | Currency | $0.20 |
| S: Cloudinary Cost | Currency | ~$0.01 |
| T: Total Cost | Currency | Sum of costs |
| U: Kling Retries | Number | Status poll count |
| V: TTS Retries | Number | Status poll count |
| W: Lip Sync Retries | Number | Status poll count |
| X: Error Message | Text | If failed |
| Y: Notes | Text | Manual notes |

### 8.3 Dashboard Metrics (Phase 7)

When Supabase dashboards are built:

**Real-Time Panel:**
- Active executions in progress
- Today's execution count
- Today's total cost
- Current success rate

**Daily Trends:**
- Executions over time (line chart)
- Cost over time (line chart)
- Success rate over time (line chart)
- Channel distribution (pie chart)

**Performance Analysis:**
- Processing time distribution (histogram)
- File size distribution (histogram)
- Compression ratio trends
- Retry frequency by service

**Cost Analysis:**
- Cost breakdown by service (stacked bar)
- Cost per execution trend
- Monthly cost projection
- Cost efficiency metrics

---

## 9. Implementation Notes

### 9.1 Data Capture Points

| Phase | Workflow Node | Data Captured |
|-------|---------------|---------------|
| 1 | Telegram Trigger | chatId, userId, fileId |
| 1 | Form Trigger | email, imageUrl |
| 1 | Workflow Configuration | All config values |
| 1 | Select Random Picasso | selectedImage |
| 1 | Submit to Kling | requestId, statusUrl |
| 1 | Check Kling Status | status, retryCount |
| 1 | Get Kling Result | videoUrl, processingTime |
| 1 | Download Kling Video | fileSize |
| 2 | Submit to ElevenLabs | requestId, characterCount |
| 2 | Get TTS Result | audioUrl, processingTime |
| 2 | Submit to LatentSync | requestId |
| 2 | Get LatentSync Result | videoUrl, processingTime |
| 3 | Cloudinary Upload | publicId, transformations |
| 3 | Cloudinary Transform | compressedSize, format |
| 1 | Archive to Google Drive | fileId, filename |
| 1 | Send to Telegram/Email | messageId, deliveryStatus |

### 9.2 Logging Strategy

**Phase 1-4 (Current):**
- Use Google Sheets for manual tracking
- Add n8n Set nodes to capture key metrics
- Export execution data weekly

**Phase 7 (Future):**
- Direct Supabase inserts from n8n
- Real-time dashboard updates
- Automated daily aggregation

### 9.3 Privacy Considerations

| Data Type | Retention | Notes |
|-----------|-----------|-------|
| User identifiers | 90 days | Telegram IDs, emails |
| Execution logs | 1 year | For debugging |
| Cost data | Permanent | For billing/analytics |
| Video files | 30 days | Cloudinary cleanup |
| Google Drive archive | Permanent | User's Drive |

---

## 10. Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-10 | 1.0 | Initial data dictionary |

---

*Document created: 2026-01-10*
*Last updated: 2026-01-10*
