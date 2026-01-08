# Picture with Picasso - Workflow Completion Checklist

**Document:** `workflow-todo-list-122125.md`
**Date:** 2025-12-21
**Workflow:** Picture with Picasso -Wan 2.5-v2.4
**Workflow ID:** `C22rlgmTijUZsUSb`
**Status:** INACTIVE - Requires fixes before activation

---

## PHASE 1: CRITICAL FIXES (Must Complete)

### 1.1 Fix Invalid Node Operations

- [ ] **Edit Image (Resize)**
  - Open node in n8n editor
  - Set "Resize Option" to: `onlyIfLarger` (recommended) or `maximumArea`
  - Save node

- [ ] **Send to Email (Gmail)**
  - Open node in n8n editor
  - Set Operation to: `send`
  - Configure recipient, subject, and body fields
  - Save node

- [ ] **Archive Video to Google Drive**
  - Open node in n8n editor
  - Set Operation to: `upload`
  - Set Parent Folder to video archive folder ID
  - Save node

- [ ] **Archive Composed Image to Drive**
  - Open node in n8n editor
  - Set Operation to: `upload`
  - Set Parent Folder to composed images folder ID
  - Save node

- [ ] **Archive User Image**
  - Open node in n8n editor
  - Set Operation to: `upload`
  - Set Parent Folder to user uploads folder ID
  - Save node

- [ ] **Archive User Image1**
  - Open node in n8n editor
  - Set Operation to: `upload`
  - Set Parent Folder to user uploads folder ID
  - Save node

### 1.2 Fix Infinite Loop Warning

- [ ] **Video Polling Loop**
  - Location: Get Videos → If → Wait → Get Videos
  - Add maximum retry counter (e.g., 30 attempts = 5 minutes at 10s intervals)
  - Add timeout condition to break loop after max attempts
  - Consider adding error notification if timeout reached

---

## PHASE 2: CREDENTIAL CONFIGURATION

### 2.1 Create/Verify n8n Credentials

- [ ] **Telegram Bot API**
  - Go to: Settings → Credentials → Add Credential
  - Type: Telegram API
  - Add Bot Token from @BotFather
  - Test connection

- [ ] **Google Drive OAuth2**
  - Go to: Settings → Credentials → Add Credential
  - Type: Google Drive OAuth2 API
  - Configure OAuth with Google Cloud Console
  - Authorize and test connection

- [ ] **Gmail OAuth2**
  - Go to: Settings → Credentials → Add Credential
  - Type: Gmail OAuth2 API
  - Configure OAuth with Google Cloud Console
  - Authorize and test connection

- [ ] **fal.ai API Key (Header Auth)**
  - Go to: Settings → Credentials → Add Credential
  - Type: Header Auth
  - Name: `fal.ai API Key`
  - Header Name: `Authorization`
  - Header Value: `Key YOUR_FAL_API_KEY`
  - Save credential

- [ ] **Update "Send to Reve Remix" Node**
  - Open node
  - Set Authentication: Generic Credential Type → Header Auth
  - Select: fal.ai API Key credential
  - Remove hardcoded Authorization from Send Headers
  - Save node

### 2.2 Set Workflow Configuration Values

- [ ] **Open "Workflow Configuration" node and set:**
  ```
  videoArchiveFolderId: [Your Google Drive Folder ID]
  userUploadedImagesFolderId: [Your Google Drive Folder ID]
  composedImagesFolderId: [Your Google Drive Folder ID]
  falApiKey: [Remove - now in credential]
  cloudinaryCloudName: [Your Cloudinary Cloud Name]
  cloudinaryApiKey: [Your Cloudinary API Key]
  duration: "10" (for longer videos)
  resolution: "720p" or "1080p"
  ```

---

## PHASE 3: NODE VERSION UPDATES (Recommended)

### 3.1 Update Outdated Nodes

- [ ] **Form Trigger** - Update from 2.3 to 2.4
- [ ] **HTTP Request nodes** - Update from 4.2 to 4.3
  - Get Videos
  - Create Video Request
  - Download Form Image
  - Check Reve Status
  - Download Composed Image
  - Upload Composed Image to Cloudinary
  - Send to Reve Remix
- [ ] **If nodes** - Update from 2.2 to 2.3
  - If
  - Route by Input Source
  - Check Reve Complete
  - Route Output
- [ ] **Merge Images for Reve** - Update from 3 to 3.2
- [ ] **Gmail (Send to Email)** - Update from 2.1 to 2.2

---

## PHASE 4: ERROR HANDLING (Recommended)

### 4.1 Add Error Handling to HTTP Nodes

- [ ] **Get Videos** - Add `retryOnFail: true`
- [ ] **Create Video Request** - Add `retryOnFail: true`
- [ ] **Send to Reve Remix** - Add `retryOnFail: true`
- [ ] **Check Reve Status** - Add `retryOnFail: true`
- [ ] **Download Composed Image** - Add `retryOnFail: true`
- [ ] **Upload Composed Image to Cloudinary** - Add `retryOnFail: true`

### 4.2 Add Error Trigger (Optional)

- [ ] Add Error Trigger node to workflow
- [ ] Connect to notification node (Telegram/Email)
- [ ] Configure error message format

---

## PHASE 5: TESTING

### 5.1 Pre-Activation Checks

- [ ] All credentials connected and tested
- [ ] All folder IDs configured
- [ ] No validation errors (run n8n validation)
- [ ] Workflow saved

### 5.2 Test Telegram Trigger

- [ ] Activate workflow
- [ ] Send test image to Telegram bot
- [ ] Verify: Image received and source detected
- [ ] Verify: Picasso image selected
- [ ] Verify: Reve Remix composition works
- [ ] Verify: Video generation completes
- [ ] Verify: Video delivered to Telegram
- [ ] Verify: Files archived to Google Drive

### 5.3 Test Form Trigger

- [ ] Open form URL in browser
- [ ] Submit test image URL and email
- [ ] Verify: Form submission received
- [ ] Verify: Image downloaded from URL
- [ ] Verify: Full pipeline completes
- [ ] Verify: Email delivered with video

### 5.4 Edge Case Testing

- [ ] Test with invalid image URL (form)
- [ ] Test with non-image file (Telegram)
- [ ] Test timeout handling (video generation)
- [ ] Test large image handling (resize)

---

## PHASE 6: ACTIVATION & MONITORING

### 6.1 Final Activation

- [ ] Review all test results
- [ ] Fix any remaining issues
- [ ] Set workflow to ACTIVE
- [ ] Document webhook URLs

### 6.2 Monitoring Setup

- [ ] Enable execution logging
- [ ] Set up error notifications
- [ ] Document expected execution time
- [ ] Create runbook for common issues

---

## QUICK REFERENCE

### Google Drive Folder IDs Needed
| Purpose | Folder ID | Status |
|---------|-----------|--------|
| Picasso Images | `1i52bCJhPJKACKsXDSzMit5rnehpXXkKi` | Configured |
| User Uploads | `1LlAN8Fbhm95x9mwEs4NAljDIkBCx-39a` | Configured |
| Composed Images | `[NEED TO SET]` | Pending |
| Video Archive | `[NEED TO SET]` | Pending |

### API Endpoints
| Service | Endpoint | Status |
|---------|----------|--------|
| Kie.ai Wan 2.5 | `api.kie.ai/api/v1/jobs/createTask` | Active |
| fal.ai Reve Remix | `fal.run/fal-ai/reve/remix` | Active |
| Cloudinary | `api.cloudinary.com/v1_1/{cloud}/image/upload` | Needs config |

### Estimated Completion Time
| Phase | Time Estimate |
|-------|---------------|
| Phase 1: Critical Fixes | 30-45 min |
| Phase 2: Credentials | 20-30 min |
| Phase 3: Updates | 15-20 min |
| Phase 4: Error Handling | 20-30 min |
| Phase 5: Testing | 45-60 min |
| Phase 6: Activation | 10-15 min |
| **Total** | **~2.5-3.5 hours** |

---

*Generated: 2025-12-21 | Based on n8n MCP validation results*
