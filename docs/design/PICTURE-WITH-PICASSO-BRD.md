# Picture with Picasso - Business Requirements Document

**Document Version:** 1.0
**Date:** 2025-11-27
**Status:** Active
**Source:** `Picture with Picasso -Wan 2.5.json`

---

## 1. Executive Summary

**Picture with Picasso** is an AI-powered video generation workflow that creates personalized videos blending user-uploaded images with Pablo Picasso artworks. Users interact via web form or Telegram bot, and receive generated videos via email or Telegram reply.

**Core Value Proposition:** Transform static images into dynamic, artistic videos featuring interaction with Picasso.

---

## 2. Business Objectives

| ID | Objective | Success Metric |
|----|-----------|----------------|
| BO-1 | Enable non-technical users to generate AI videos | < 3 user inputs required |
| BO-2 | Dual-channel input accessibility | Form + Telegram operational |
| BO-3 | Automated delivery to user's preferred channel | 100% routing accuracy |
| BO-4 | Minimal operational overhead | Fully automated workflow |

---

## 3. Scope
### 3.1 In Scope

- Web form input (image upload + email)
- Telegram bot input (message trigger)
- Picasso image library management (Google Drive)
- AI video generation via Kie.ai (Wan 2.5 model)
- Email delivery (Gmail)
- Telegram reply delivery
- Form submission logging (n8n Data Table)

### 3.2 Out of Scope

- User authentication/accounts
- Payment processing
- Video editing/customization UI
- Analytics dashboard
- Multi-language support

---

## 4. Functional Requirements

### 4.1 Input Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1 | System SHALL accept image uploads via web form | P0 |
| FR-2 | System SHALL capture user email from web form | P0 |
| FR-3 | System SHALL accept Telegram messages as trigger | P0 |
| FR-4 | System SHALL capture Telegram chat ID for reply | P0 |

### 4.2 Processing Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-5 | System SHALL cycle through Picasso images (round-robin) | P0 |
| FR-6 | System SHALL generate video prompt combining user + Picasso context | P0 |
| FR-7 | System SHALL submit video generation request to Kie.ai API | P0 |
| FR-8 | System SHALL poll for video completion status | P0 |
| FR-9 | System SHALL wait 1 minute between poll attempts | P1 |

### 4.3 Output Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-10 | System SHALL route output based on input source | P0 |
| FR-11 | System SHALL send video via Telegram if source=telegram | P0 |
| FR-12 | System SHALL send video via Email if source=form | P0 |
| FR-13 | System SHALL log form submissions to data table | P1 |

---

## 5. Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-1 | Video generation completion | < 5 minutes |
| NFR-2 | System availability | 99% uptime |
| NFR-3 | Concurrent request handling | 5 simultaneous |
| NFR-4 | API error recovery | Automatic retry loop |

---

## 6. User Stories

### US-1: Web Form User
> As a **website visitor**, I want to **upload my photo and provide my email**, so that I can **receive an AI video of me with Picasso**.

**Acceptance Criteria:**
- [ ] Form displays image upload field
- [ ] Form displays email input field
- [ ] Form submission triggers workflow
- [ ] User receives video at provided email

### US-2: Telegram User
> As a **Telegram user**, I want to **send a message to the bot**, so that I can **receive an AI video reply**.

**Acceptance Criteria:**
- [ ] Bot responds to incoming messages
- [ ] Bot generates video
- [ ] Bot sends video as reply to same chat

---

## 7. Technical Architecture

### 7.1 Workflow Diagram

```
                    ┌─────────────────┐
                    │   Form Trigger  │
                    │  (Web Upload)   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐        ┌─────────────────┐
                    │    Workflow     │        │ Store Form      │
                    │  Configuration  │───────▶│ Responses       │
                    └────────┬────────┘        └─────────────────┘
                             │
┌─────────────────┐          │
│Telegram Trigger │          │
│   (Bot Input)   │          │
└────────┬────────┘          │
         │                   │
         ▼                   │
┌─────────────────┐          │
│ Prepare Telegram│          │
│     Input       │          │
└────────┬────────┘          │
         │                   │
         └───────────┬───────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Get Picasso     │
            │ Images (Drive)  │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Select Random   │
            │ Picasso Image   │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Download        │
            │ Picasso Image   │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Prepare Video   │
            │ Prompt          │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Create Video    │
            │ Request (Kie.ai)│
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Get Videos      │◀──────────┐
            │ (Poll Status)   │           │
            └────────┬────────┘           │
                     │                    │
                     ▼                    │
            ┌─────────────────┐           │
            │ If Success?     │───No─────▶│ Wait 1 min
            └────────┬────────┘           │
                     │ Yes                │
                     ▼                    │
            ┌─────────────────┐           │
            │ Route Output    │           │
            └────────┬────────┘           │
                     │
         ┌───────────┴───────────┐
         │ Telegram              │ Form
         ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│ Send to         │     │ Send to         │
│ Telegram        │     │ Email           │
└─────────────────┘     └─────────────────┘
```

### 7.2 Node Inventory

| Node Name | Type | Purpose |
|-----------|------|---------|
| Form Trigger | n8n-nodes-base.formTrigger | Web form entry point |
| Telegram Trigger | n8n-nodes-base.telegramTrigger | Bot message entry point |
| Workflow Configuration | n8n-nodes-base.set | Normalize form input |
| Prepare Telegram Input | n8n-nodes-base.set | Normalize Telegram input |
| Store Form Responses | n8n-nodes-base.dataTable | Log submissions |
| Get Picasso Images | n8n-nodes-base.googleDrive | List Drive folder contents |
| Select Random Picasso Image | n8n-nodes-base.code | Round-robin selection |
| Download Picasso Image | n8n-nodes-base.googleDrive | Fetch image binary |
| Prepare Video Prompt | n8n-nodes-base.set | Assemble Kie.ai payload |
| Create Video Request | n8n-nodes-base.httpRequest | POST to Kie.ai |
| Get Videos | n8n-nodes-base.httpRequest | GET poll status |
| If | n8n-nodes-base.if | Check success state |
| Wait | n8n-nodes-base.wait | 1 minute delay |
| Route Output | n8n-nodes-base.if | Source-based routing |
| Send to Telegram | n8n-nodes-base.telegram | Video reply |
| Send to Email | n8n-nodes-base.gmail | Email with attachment |

---

## 8. Data Flow

### 8.1 Input Schema (Normalized)

```json
{
  "inputSource": "form | telegram",
  "userEmail": "string (form only)",
  "chatId": "string (telegram only)",
  "duration": "5",
  "resolution": "720p",
  "optimizePrompt": true
}
```

### 8.2 Video Prompt (Kie.ai Payload)

```json
{
  "model": "wan/2-5-image-to-video",
  "input": {
    "prompt": "Add the people from the uploaded image into the environment where Picasso is in this picture. Make them horse around and be chummy with Picasso in this setting. Keep the artistic style and atmosphere of the original Picasso artwork.",
    "image_url": "<binary_data>",
    "duration": "5",
    "resolution": "720p",
    "enable_prompt_expansion": true
  }
}
```

### 8.3 Kie.ai Response Path

```
Response → data.state === "success"
         → data.resultJson (parse)
         → resultUrls[0] → video URL
```

---

## 9. Integration Requirements

### 9.1 External Services

| Service | Purpose | Credential Type |
|---------|---------|-----------------|
| Telegram Bot API | Input/Output | Bot Token |
| Google Drive API | Image storage | Service Account |
| Gmail API | Email delivery | Service Account |
| Kie.ai API | Video generation | HTTP Header Auth |

### 9.2 Kie.ai API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `api.kie.ai/api/v1/jobs/createTask` | POST | Submit video job |
| `api.kie.ai/api/v1/jobs/recordInfo?taskId={id}` | GET | Poll job status |

### 9.3 Credential Configuration

**Telegram Bot:**
- Create via @BotFather
- Store token in n8n Telegram API credential

**Google Service Account:**
- Create in Google Cloud Console
- Enable Drive API + Gmail API
- Download JSON key
- Share Drive folder with service account email

**Kie.ai:**
- Sign up at kie.ai
- Generate API key
- Configure as HTTP Header Auth:
  - Header: `Authorization`
  - Value: `Bearer <API_KEY>`

---

## 10. Acceptance Criteria

### 10.1 Form Flow
- [ ] Form accessible via n8n form URL
- [ ] Image upload functional
- [ ] Email field validates format
- [ ] Submission triggers workflow
- [ ] Video delivered to email within 5 minutes

### 10.2 Telegram Flow
- [ ] Bot responds to any message
- [ ] Video generated and sent as reply
- [ ] Reply appears in same chat

### 10.3 Video Generation
- [ ] Kie.ai job created successfully
- [ ] Polling loop retries until success
- [ ] Video URL extracted from response

---

## 11. Dependencies

| Dependency | Type | Risk |
|------------|------|------|
| Kie.ai API availability | External | Medium |
| Google Drive folder with Picasso images | Content | Low |
| Telegram Bot configuration | Setup | Low |
| Google Service Account permissions | Setup | Medium |

---

## 12. Cost Estimates

### 12.1 Kie.ai Video Generation

| Resolution | Cost per Second | 5s Video | 10s Video |
|------------|-----------------|----------|-----------|
| 480p | $0.05 | $0.25 | $0.50 |
| 720p | $0.06 | $0.30 | $0.60 |
| 1080p | $0.10 | $0.50 | $1.00 |

**Default Configuration:** 720p, 5 seconds = **$0.30 per video**

### 12.2 Other Services

| Service | Cost |
|---------|------|
| n8n (self-hosted) | Infrastructure only |
| n8n Cloud | Per execution pricing |
| Google Drive | Free tier sufficient |
| Gmail API | Free tier sufficient |
| Telegram Bot | Free |

---

## 13. Configuration Checklist

### Pre-Deployment

- [ ] Google Drive folder created with Picasso images
- [ ] Google Drive folder ID copied to `Get Picasso Images` node
- [ ] Google Service Account created and JSON key downloaded
- [ ] Service Account email shared on Drive folder
- [ ] Gmail API enabled in Google Cloud Console
- [ ] Telegram Bot created via @BotFather
- [ ] Telegram Bot token configured in n8n
- [ ] Kie.ai account created
- [ ] Kie.ai API key configured as HTTP Header Auth

### Post-Deployment

- [ ] Form URL tested
- [ ] Telegram Bot responds to messages
- [ ] Video generation completes
- [ ] Email delivery verified
- [ ] Telegram reply verified

---

## 14. Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-27 | Claude | Initial BRD from workflow JSON |

---

## Appendix A: Sticky Note Reference (from JSON)

The workflow includes embedded documentation sticky notes:

1. **Workflow Overview** - General description
2. **Telegram Setup** - Bot configuration steps
3. **Google Drive Setup** - Folder and service account
4. **Gmail Setup** - Email API configuration
5. **Kie.ai API Setup** - Video generation credentials

---

*End of Document*
