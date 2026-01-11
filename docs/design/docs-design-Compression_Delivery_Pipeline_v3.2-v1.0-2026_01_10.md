# Compression & Delivery Pipeline Design - Picture with Picasso v3.2

**Version:** v1.0-2026-01-10
**Purpose:** Video compression via Cloudinary and multi-channel delivery with archival
**Prerequisite:** Audio Pipeline v3.1 (docs-design-Audio_Pipeline_v3.1-v1.0-2026_01_10.md)

---

## Design Overview

### Objective
After audio is added (v3.1), compress the video to reduce file size from 20-30 MB to 5-10 MB, then deliver as an actual file (not link) to the appropriate channel with Google Drive archival.

### Architecture Summary
```
v3.1 Output (With Audio, ~25 MB)
       ↓
[Cloudinary Upload + Compression]
       ↓
[Download Compressed Binary]
       ↓
[Generate Filenames]
       ↓
[Parallel Delivery]
    ├── Telegram (file) ─ based on input method
    ├── Email (attachment) ─ based on input method
    └── Google Drive (archive) ─ always
```

### Problem Statement

| Issue | Root Cause | Solution |
|-------|------------|----------|
| File size 18-25 MB (target 5-10 MB) | Kling O1 has no quality/resolution parameters | Cloudinary post-processing |
| Telegram mobile display distortion | Known Telegram client bug #30068 | Explicit dimensions + metadata normalization |
| No delivery as actual file | Current workflow sends URLs | Download binary, send as attachment |
| No permanent archive | Videos stored temporarily | Google Drive archival |

### Research Sources

| Finding | Source |
|---------|--------|
| Kling O1 has no quality parameters | [fal.ai API Docs](https://fal.ai/models/fal-ai/kling-video/o1/reference-to-video/api) |
| Cloudinary q_auto:eco reduces 40-60% | [Cloudinary Video Optimization](https://cloudinary.com/documentation/video_optimization) |
| Telegram 9:16 display bug confirmed | [GitHub Issue #30068](https://github.com/telegramdesktop/tdesktop/issues/30068) |

---

## Pipeline Specifications

### Node Count
**Total New Nodes: 8-10**

| Stage | Nodes | Purpose |
|-------|-------|---------|
| Cloudinary Upload | 1 | Upload with eager transformation |
| Download Binary | 1 | Get compressed file as binary |
| Filename Generation | 1 | Generate internal + external names |
| Delivery Routing | 1 | Switch based on input method |
| Telegram Send | 1 | Send file attachment |
| Email Send | 1 | Send file attachment |
| Google Drive Upload | 1 | Archive with internal filename |
| Metadata Logging | 1-2 | Optional: log to Google Sheets |

---

## Stage 1: Cloudinary Compression

### Node: Upload to Cloudinary
**Type:** HTTP Request
**Name:** `Upload to Cloudinary`
**Position:** After `Download Final Video` (from v3.1)

**Configuration:**
| Field | Value |
|-------|-------|
| Method | POST |
| URL | `https://api.cloudinary.com/v1_1/{{ $('Workflow Configuration').item.json.cloudinaryCloudName }}/video/upload` |
| Authentication | None (uses signed params) |
| Send Body | On |
| Body Type | Form-Data |

**Form-Data Fields:**
| Field | Value |
|-------|-------|
| `file` | `={{ $json.data }}` (binary from previous node) OR URL |
| `upload_preset` | `pwp_video_compress` |
| `eager` | `q_auto:eco,f_auto:video,c_limit,w_1080` |
| `eager_async` | `false` |
| `resource_type` | `video` |

**Alternative: Signed Upload (if preset not used):**
| Field | Value |
|-------|-------|
| `api_key` | `{{ $('Workflow Configuration').item.json.cloudinaryApiKey }}` |
| `timestamp` | `={{ Math.floor(Date.now() / 1000) }}` |
| `signature` | `={{ computed signature }}` |

**Transformation Parameters Explained:**
| Parameter | Effect | File Size Impact |
|-----------|--------|------------------|
| `q_auto:eco` | Aggressive compression, good quality | -40% to -60% |
| `f_auto:video` | Auto-select best codec (VP9/HEVC/H.264) | -20% to -30% |
| `c_limit,w_1080` | Constrain width to 1080px, maintain aspect | -10% to -20% |

**Expected Response:**
```json
{
  "public_id": "pwp/video_abc123",
  "secure_url": "https://res.cloudinary.com/.../q_auto:eco.../video.mp4",
  "eager": [
    {
      "secure_url": "https://res.cloudinary.com/.../transformed.mp4",
      "width": 1080,
      "height": 1920,
      "bytes": 6500000
    }
  ],
  "width": 1080,
  "height": 1920,
  "bytes": 6500000,
  "format": "mp4",
  "duration": 10.5
}
```

---

## Stage 2: Download Compressed Binary

### Node: Download Compressed Video
**Type:** HTTP Request
**Name:** `Download Compressed Video`
**Position:** After `Upload to Cloudinary`

**Configuration:**
| Field | Value |
|-------|-------|
| Method | GET |
| URL | `={{ $json.eager[0].secure_url }}` |
| Response Format | File |
| Put Output in Field | `compressedVideo` |

**Output:**
Binary video data stored in `compressedVideo` field

---

## Stage 3: Filename Generation

### Node: Generate Filenames
**Type:** Set
**Name:** `Generate Filenames`
**Position:** After `Download Compressed Video`

**Fields:**

#### Internal Archive Filename
```javascript
internalFilename: {{
  'PwP_' +
  $('Route by Trigger').item.json.inputMethod + '_' +
  $now.format('yyyy-MM-dd') + '_' +
  $now.format('HH-mm-ss') + '_' +
  $execution.id.substring(0, 6) + '.mp4'
}}
```

**Example Output:** `PwP_telegram_2026-01-10_14-32-05_abc123.mp4`

#### External User-Facing Filename
```javascript
externalFilename: {{
  'Picasso-Portrait_' +
  $execution.id.substring(0, 4).toUpperCase() + '.mp4'
}}
```

**Example Output:** `Picasso-Portrait_7K9M.mp4`

#### Video Metadata
```javascript
videoMetadata: {{
  JSON.stringify({
    executionId: $execution.id,
    timestamp: $now.toISO(),
    inputMethod: $('Route by Trigger').item.json.inputMethod,
    userId: $('Route by Trigger').item.json.userId,
    originalSize: $('Upload to Cloudinary').item.json.bytes,
    compressedSize: $json.eager[0].bytes,
    compressionRatio: Math.round((1 - $json.eager[0].bytes / $('Upload to Cloudinary').item.json.bytes) * 100) + '%',
    width: $json.eager[0].width,
    height: $json.eager[0].height,
    duration: $json.duration
  })
}}
```

---

## Stage 4: Delivery Routing

### Node: Route Delivery
**Type:** Switch
**Name:** `Route Delivery`
**Position:** After `Generate Filenames`

**Routing Rules:**
| Condition | Output |
|-----------|--------|
| `{{ $('Route by Trigger').item.json.inputMethod }}` equals `telegram` | Telegram branch |
| `{{ $('Route by Trigger').item.json.inputMethod }}` equals `email` | Email branch |

**Note:** Google Drive upload happens for ALL paths (not conditional)

---

## Stage 5: Telegram Delivery

### Node: Send to Telegram
**Type:** Telegram
**Name:** `Send Video to Telegram`
**Position:** Telegram branch of `Route Delivery`

**Configuration:**
| Field | Value |
|-------|-------|
| Operation | Send Video |
| Chat ID | `={{ $('Route by Trigger').item.json.telegramChatId }}` |
| Binary Property | `compressedVideo` |
| File Name | `={{ $json.externalFilename }}` |
| Width | `={{ $('Upload to Cloudinary').item.json.eager[0].width }}` |
| Height | `={{ $('Upload to Cloudinary').item.json.eager[0].height }}` |
| Supports Streaming | `true` |
| Caption | `Your Picture with Picasso portrait is ready!` |

**Telegram Display Fix:**
Explicitly passing `width` and `height` helps Telegram render the correct aspect ratio.

---

## Stage 6: Email Delivery

### Node: Send via Email
**Type:** Gmail (or SMTP)
**Name:** `Send Video via Email`
**Position:** Email branch of `Route Delivery`

**Configuration:**
| Field | Value |
|-------|-------|
| Operation | Send |
| To | `={{ $('Route by Trigger').item.json.userEmail }}` |
| Subject | `Your Picture with Picasso Portrait` |
| Email Type | HTML |
| Attachments | Binary Property: `compressedVideo` |
| Attachment Name | `={{ $json.externalFilename }}` |

**Email Body:**
```html
<p>Hello!</p>
<p>Your Picture with Picasso portrait is attached.</p>
<p>Thank you for using Picture with Picasso!</p>
```

---

## Stage 7: Google Drive Archive

### Node: Archive to Google Drive
**Type:** Google Drive
**Name:** `Archive to Google Drive`
**Position:** Parallel to both Telegram and Email (always executes)

**Configuration:**
| Field | Value |
|-------|-------|
| Operation | Upload |
| File Name | `={{ $json.internalFilename }}` |
| Binary Property | `compressedVideo` |
| Parent Folder | `{{ $('Workflow Configuration').item.json.googleDriveArchiveFolder }}` |

**Folder Structure Recommendation:**
```
Picture with Picasso Archive/
    2026/
        01/
            PwP_telegram_2026-01-10_14-32-05_abc123.mp4
            PwP_email_2026-01-10_16-45-22_def456.mp4
```

---

## Stage 8: Metadata Logging (Optional)

### Node: Log to Google Sheets
**Type:** Google Sheets
**Name:** `Log Video Metadata`
**Position:** After Google Drive upload

**Configuration:**
| Field | Value |
|-------|-------|
| Operation | Append Row |
| Spreadsheet | `Picture with Picasso - Video Log` |
| Sheet | `Archive Log` |

**Row Data:**
| Column | Value |
|--------|-------|
| A: Execution ID | `={{ $execution.id }}` |
| B: Date | `={{ $now.format('yyyy-MM-dd') }}` |
| C: Time | `={{ $now.format('HH:mm:ss') }}` |
| D: Input Method | `={{ $json.inputMethod }}` |
| E: User ID | `={{ $json.userId }}` |
| F: Original Size (MB) | `={{ Math.round($json.originalSize / 1024 / 1024 * 100) / 100 }}` |
| G: Compressed Size (MB) | `={{ Math.round($json.compressedSize / 1024 / 1024 * 100) / 100 }}` |
| H: Compression % | `={{ $json.compressionRatio }}` |
| I: Google Drive Path | `={{ $json.googleDriveUrl }}` |
| J: Status | `Delivered` |

---

## Connection Diagram

```
                    ┌─────────────────────────┐
                    │  Download Final Video   │
                    │  (from v3.1 Audio)      │
                    └───────────┬─────────────┘
                                │
                                ▼
                    ┌─────────────────────────┐
                    │  Upload to Cloudinary   │
                    │  (q_auto:eco,f_auto)    │
                    └───────────┬─────────────┘
                                │
                                ▼
                    ┌─────────────────────────┐
                    │  Download Compressed    │
                    │  Video (Binary)         │
                    └───────────┬─────────────┘
                                │
                                ▼
                    ┌─────────────────────────┐
                    │  Generate Filenames     │
                    │  (Internal + External)  │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            │                   │                   │
            ▼                   ▼                   ▼
  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
  │  Route Delivery │ │  Archive to     │ │  Log to Google  │
  │  (Switch)       │ │  Google Drive   │ │  Sheets         │
  └────────┬────────┘ │  (ALWAYS)       │ │  (Optional)     │
           │          └─────────────────┘ └─────────────────┘
     ┌─────┴─────┐
     │           │
     ▼           ▼
┌─────────┐ ┌─────────┐
│Telegram │ │  Email  │
│  Send   │ │  Send   │
│ (File)  │ │ (File)  │
└─────────┘ └─────────┘
```

---

## File Naming Convention

### Internal Archive Name (Google Drive)

**Format:**
```
PwP_{inputMethod}_{date}_{time}_{executionId6}.mp4
```

**Components:**
| Component | Description | Example |
|-----------|-------------|---------|
| `PwP` | Project prefix | PwP |
| `inputMethod` | How user initiated | telegram, email |
| `date` | ISO date (sortable) | 2026-01-10 |
| `time` | HH-MM-SS | 14-32-05 |
| `executionId6` | First 6 chars of n8n execution ID | abc123 |

**Examples:**
- `PwP_telegram_2026-01-10_14-32-05_abc123.mp4`
- `PwP_email_2026-01-10_16-45-22_def456.mp4`

### External User-Facing Name (Telegram/Email)

**Format:**
```
Picasso-Portrait_{shortId4}.mp4
```

**Components:**
| Component | Description | Example |
|-----------|-------------|---------|
| `Picasso-Portrait` | Brand identifier | Picasso-Portrait |
| `shortId4` | 4-char uppercase from execution ID | 7K9M |

**Examples:**
- `Picasso-Portrait_7K9M.mp4`
- `Picasso-Portrait_A3B2.mp4`

**Rationale:**
- Short, memorable, professional
- Unique identifier prevents confusion
- Brand recognition without clutter

---

## Configuration Requirements

### Workflow Configuration Node Additions

```json
{
  "cloudinaryCloudName": "your_cloud_name",
  "cloudinaryApiKey": "your_api_key",
  "cloudinaryApiSecret": "your_api_secret",
  "cloudinaryUploadPreset": "pwp_video_compress",
  "googleDriveArchiveFolder": "folder_id_here",
  "googleSheetsLogId": "spreadsheet_id_here"
}
```

### Cloudinary Setup Requirements

1. **Create Upload Preset** (recommended approach):
   - Name: `pwp_video_compress`
   - Eager transformation: `q_auto:eco,f_auto:video,c_limit,w_1080`
   - Unique filename: On
   - Delivery type: Upload

2. **Alternative: Signed Uploads**
   - Store API key and secret
   - Generate signature per request

---

## Expected Results

### File Size Reduction

| Stage | Expected Size |
|-------|---------------|
| Kling O1 output (silent) | 18-25 MB |
| After v3.1 Audio | 20-30 MB |
| After v3.2 Cloudinary | **5-10 MB** |

**Compression Breakdown:**
| Transformation | Reduction |
|----------------|-----------|
| q_auto:eco | 40-60% |
| f_auto (VP9/HEVC) | 20-30% additional |
| c_limit,w_1080 | 10-20% additional |
| **Total** | **60-80%** |

### Telegram Display Improvement

| Issue | Mitigation |
|-------|------------|
| Vertical video squeezed | Explicit width/height parameters |
| Aspect ratio metadata | Cloudinary normalizes SAR/DAR |
| Preview distortion | Client-side bug; full playback correct |

---

## Error Handling

### Cloudinary Upload Failures
- Retry: 3 attempts with 5-second delay
- Fallback: Send uncompressed video (larger file)
- Log error to notification system

### Google Drive Upload Failures
- Retry: 3 attempts
- Fallback: Continue with delivery, log failure
- Video is still delivered to user

### Delivery Failures
- Telegram: Retry 2x, then notify admin
- Email: Retry 2x, then notify admin
- Log all failures to Google Sheets

---

## Cost Estimate

### Per Video

| Component | Cost |
|-----------|------|
| Kling O1 | $1.00 |
| ElevenLabs TTS (v3.1) | ~$0.05 |
| LatentSync (v3.1) | ~$0.20 |
| Cloudinary | ~$0.01 (transformation credit) |
| Google Drive | Free (within storage limits) |
| **Total v3.2** | **~$1.26** |

### Monthly (Estimated 100 videos)
- Cloudinary: Free tier covers 25 credits, then ~$45/month
- Google Drive: Free tier likely sufficient

---

## Implementation Phases

### Phase 1: Cloudinary Integration
- Create upload preset
- Add upload node
- Verify compression results

### Phase 2: Binary Download + Filenames
- Download compressed binary
- Implement filename generation
- Test both naming conventions

### Phase 3: Delivery Routing
- Update Telegram send node (file, not URL)
- Update Email send node (attachment)
- Test both delivery paths

### Phase 4: Google Drive Archive
- Configure archive folder
- Add upload node
- Verify file naming

### Phase 5: Logging (Optional)
- Set up Google Sheets log
- Add logging node
- Test audit trail

---

## Testing Plan

### Unit Tests
1. Cloudinary upload with test video
2. Filename generation logic
3. Binary download from Cloudinary URL

### Integration Tests
1. Full pipeline: Audio → Compress → Telegram
2. Full pipeline: Audio → Compress → Email
3. Google Drive archive verification

### Quality Checks
- [ ] File size reduced to target (5-10 MB)
- [ ] Video quality acceptable after compression
- [ ] Audio preserved after Cloudinary processing
- [ ] Telegram displays without distortion
- [ ] Email attachment opens correctly
- [ ] Google Drive file accessible
- [ ] Filenames follow convention
- [ ] Metadata log accurate

---

## Dependencies

### External Services
| Service | Purpose | Credentials Needed |
|---------|---------|-------------------|
| Cloudinary | Video compression | Cloud name, API key, API secret |
| Google Drive | Archive storage | OAuth credentials |
| Google Sheets | Metadata logging | OAuth credentials |
| Telegram | Delivery channel | Bot token (existing) |
| Gmail | Delivery channel | OAuth credentials (existing) |

### Workflow Dependencies
| Dependency | Location |
|------------|----------|
| Audio Pipeline v3.1 | Must complete before v3.2 |
| Route by Trigger node | Provides inputMethod |
| Workflow Configuration | Stores credentials/settings |

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-10 | 1.0 | Initial design |

---

## Related Documents

| Document | Purpose |
|----------|---------|
| `docs-design-Audio_Pipeline_v3.1-v1.0-2026_01_10.md` | Audio integration design |
| `docs-research-Lip_Sync_Audio_Pipeline-v1.0-2026_01_10.md` | Lip sync research |
| `PICTURE-WITH-PICASSO-WORKFLOW-PLAN.md` | Overall workflow plan |

---

*Design completed: 2026-01-10*
