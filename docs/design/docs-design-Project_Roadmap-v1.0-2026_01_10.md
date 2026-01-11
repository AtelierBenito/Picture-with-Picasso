# Picture with Picasso - Project Roadmap

**Version:** v1.0-2026-01-10
**Document Type:** Master Reference
**Owner:** Atelier Benito / RobotBird.AI
**Status:** Active

---

## Executive Summary

Picture with Picasso is an AI-powered video generation experience that creates personalized "meeting Picasso" moments. Users submit a photo and receive a video of themselves being greeted by Pablo Picasso in his studio.

**Current State:** Phase 1 Complete (Core MVP)
**Target State:** Phase 4 (Enhanced Delivery with Branding)
**Future Vision:** SaaS product on RobotBird.io with social media integration

---

## Project Identity

| Element | Value |
|---------|-------|
| **Project Name** | Picture with Picasso |
| **Brand** | Atelier Benito |
| **Producer** | RobotBird.AI |
| **Attribution** | "An Atelier Benito joint, produced by RobotBird.AI" |

---

## Phase Summary Table

| Phase | Name | Status | What It Achieves | Benefit |
|-------|------|--------|------------------|---------|
| **1** | Core MVP | ✅ Complete | Silent video generation from user photo using Kling O1, delivered via Telegram/Email | Proves concept works; users can receive personalized Picasso videos |
| **2** | Audio Integration | 📝 Designed | Adds Picasso voice (Spanish accent) with synchronized lip movements | Transforms silent video into immersive experience; emotional connection |
| **3** | Compression & Delivery | 📝 Designed | Reduces file size 60-80%, delivers actual files (not links), archives to Google Drive | Professional delivery; manageable file sizes; complete audit trail |
| **4** | Enhanced Delivery | 🎯 Target | Branded messaging, hashtag-ready captions, platform-optimized content | Social-media-ready output; brand recognition; user can share immediately |
| **5** | Social Posting | 📋 Future | One-click posting to Instagram, TikTok, X, Facebook with watermark | Viral potential; reduced friction; brand exposure on every share |
| **6** | Web Frontend | 📋 Future | RobotBird.io web interface with Vercel hosting | Broader reach beyond messaging apps; professional web presence |
| **7** | SaaS Foundation | 📋 Future | User auth, Stripe payments, subscription tiers | Revenue generation; scalable business model |
| **8** | Mobile & Messaging | 📋 Future | iOS/Android apps, WhatsApp, additional channels | Maximum accessibility; native mobile experience |
| **9** | Scale & Enterprise | 📋 Future | Multi-tenant, analytics, white-label options | Enterprise sales; high-volume processing; B2B opportunities |

---

## Work Effort by Phase

| Phase | Name | Effort (Hours) | Scope | Key Activities |
|-------|------|----------------|-------|----------------|
| **1** | Core MVP | ✅ Complete (~40 hrs) | CURRENT | Workflow setup, Kling integration, Telegram/Email, archival |
| **2** | Audio Integration | 12-16 hrs | CURRENT | 11 nodes, TTS integration, lip sync, testing |
| **3** | Compression & Delivery | 8-12 hrs | CURRENT | 8-10 nodes, Cloudinary setup, filename logic, delivery |
| **4** | Enhanced Delivery | 4-6 hrs | CURRENT | Message templates, hashtags, branding text |
| **5** | Social Posting | 20-30 hrs | FUTURE | API integrations (4 platforms), watermark, buttons |
| **6** | Web Frontend | 40-60 hrs | FUTURE | Next.js app, UI/UX, Vercel deployment, v0 templates |
| **7** | SaaS Foundation | 60-80 hrs | FUTURE | Supabase backend, auth, Stripe, admin dashboard |
| **8** | Mobile & Messaging | 80-120 hrs | FUTURE | React Native apps, WhatsApp, app store submission |
| **9** | Scale & Enterprise | 60-100 hrs | FUTURE | Multi-tenant, analytics, white-label, SLAs |

### Current Scope Effort Summary (Phases 1-4)

| Phase | Status | Hours | Cumulative |
|-------|--------|-------|------------|
| Phase 1 | ✅ Complete | ~40 | 40 |
| Phase 2 | 📝 Ready to Build | 12-16 | 52-56 |
| Phase 3 | 📝 Ready to Build | 8-12 | 60-68 |
| Phase 4 | 🎯 To Design | 4-6 | 64-74 |
| **Total to Phase 4** | | **24-34 hrs remaining** | **64-74 hrs total** |

### Future Scope Effort Summary (Phases 5-9)

| Phase | Hours | Cumulative from Phase 5 |
|-------|-------|-------------------------|
| Phase 5 | 20-30 | 20-30 |
| Phase 6 | 40-60 | 60-90 |
| Phase 7 | 60-80 | 120-170 |
| Phase 8 | 80-120 | 200-290 |
| Phase 9 | 60-100 | 260-390 |
| **Total Future** | **260-390 hrs** | |

---

## Credentials & Accounts Required by Phase

### Phase 1: Core MVP ✅ HAVE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| fal.ai API Key | Kling O1 video generation | ✅ Have | Stored in workflow config |
| Telegram Bot Token | Telegram Bot API | ✅ Have | @YourBot |
| Gmail OAuth | Email input/output | ✅ Have | OAuth credentials |
| Google Drive OAuth | Archive storage | ✅ Have | Same Google account |

### Phase 2: Audio Integration 📝 NEED TO VERIFY

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| fal.ai API Key | ElevenLabs TTS, LatentSync | ✅ Have | Same key covers all fal.ai models |
| ElevenLabs Voice ID | Picasso voice | ⚠️ Need | Create/select Spanish-accented voice |

### Phase 3: Compression & Delivery 📝 NEED

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| Cloudinary Cloud Name | Media storage | ⚠️ Need | Free tier available |
| Cloudinary API Key | Upload API | ⚠️ Need | From Cloudinary dashboard |
| Cloudinary API Secret | Signed uploads | ⚠️ Need | Keep secure |
| Google Sheets OAuth | Metadata logging | ✅ Have | Same Google account |

### Phase 4: Enhanced Delivery 🎯 NO NEW CREDENTIALS

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| (None required) | Text formatting only | ✅ N/A | Uses existing services |

### Phase 5: Social Posting 📋 FUTURE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| Facebook Business Account | Instagram + Facebook APIs | 📋 Future | Required for both platforms |
| Instagram Graph API Token | Reels posting | 📋 Future | Via Facebook Business |
| Facebook Graph API Token | Feed + Reels posting | 📋 Future | Via Facebook Business |
| TikTok Developer App | Content Posting API | 📋 Future | 2-4 week approval process |
| X Developer Account | X API v2 | 📋 Future | Apply at developer.twitter.com |
| X API Keys | Tweet posting | 📋 Future | Bearer token + OAuth |

### Phase 6: Web Frontend 📋 FUTURE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| Vercel Account | Web hosting | 📋 Future | Free tier available |
| Vercel API Token | Deployment | 📋 Future | For CI/CD |
| Domain DNS | RobotBird.io subdomain | 📋 Future | Configure in Vercel |

### Phase 7: SaaS Foundation 📋 FUTURE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| Supabase Project | Database + Auth | 📋 Future | Free tier available |
| Supabase API Keys | Backend access | 📋 Future | Anon + Service role keys |
| Stripe Account | Payments | 📋 Future | Business verification required |
| Stripe API Keys | Payment processing | 📋 Future | Publishable + Secret keys |
| Stripe Webhook Secret | Payment events | 📋 Future | For subscription handling |

### Phase 8: Mobile & Messaging 📋 FUTURE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| Apple Developer Account | iOS App Store | 📋 Future | $99/year |
| App Store Connect | iOS distribution | 📋 Future | Part of Apple Developer |
| Google Play Console | Android distribution | 📋 Future | $25 one-time |
| WhatsApp Business API | Messaging | 📋 Future | Via Meta Business Suite |
| Twilio Account | SMS (optional) | 📋 Future | Pay-per-use |

### Phase 9: Scale & Enterprise 📋 FUTURE

| Credential | Service | Status | Notes |
|------------|---------|--------|-------|
| (Scaling existing services) | Higher tier plans | 📋 Future | Upgrade as needed |

### Credentials Summary

| Phase | New Credentials Needed | Estimated Setup Time |
|-------|------------------------|---------------------|
| 1 | ✅ Complete | Done |
| 2 | 1 (ElevenLabs Voice ID) | 30 min |
| 3 | 3 (Cloudinary) | 1 hour |
| 4 | 0 | N/A |
| 5 | 5 (Social platforms) | 2-4 weeks (approvals) |
| 6 | 2 (Vercel) | 1 hour |
| 7 | 4 (Supabase, Stripe) | 2-3 hours |
| 8 | 4 (App stores, WhatsApp) | 1-2 weeks (approvals) |
| 9 | 0 (scaling existing) | N/A |

---

## Current Scope: Phases 1-4

### Phase 1: Core MVP ✅ COMPLETE

**Objective:** Prove the concept works end-to-end

**Deliverables:**
| Component | Description | Status |
|-----------|-------------|--------|
| Kling O1 Integration | Reference-to-video generation via fal.ai | ✅ |
| Telegram Input | User sends photo via Telegram bot | ✅ |
| Email Input | User sends photo via email | ✅ |
| Video Delivery | Send generated video back to user | ✅ |
| Google Drive Archive | Store all videos with timestamps | ✅ |

**Achieved:**
- Users can submit photos and receive personalized Picasso videos
- Multiple input channels (Telegram, Email)
- Basic archival for record-keeping

**Limitations:**
- Videos are silent
- Large file sizes (18-25 MB)
- Delivered as URLs, not files
- No branding or social-ready formatting

---

### Phase 2: Audio Integration (v3.1) 📝 DESIGNED

**Objective:** Add voice and lip sync for immersive experience

**Design Document:** `docs-design-Audio_Pipeline_v3.1-v1.0-2026_01_10.md`

**Deliverables:**
| Component | Description | Status |
|-----------|-------------|--------|
| ElevenLabs TTS | Spanish-accented Picasso voice | 📝 Designed |
| LatentSync | Lip synchronization to audio | 📝 Designed |
| Dialog Script | Welcoming greeting in Picasso character | 📝 Designed |
| Error Handling | Graceful fallback to silent video | 📝 Designed |

**New Nodes:** 11

**What It Achieves:**
- Picasso speaks with authentic Spanish accent
- Lip movements match the audio
- Emotionally engaging experience

**Benefit:**
- Transforms from novelty to memorable experience
- Shareable "wow" factor
- Differentiates from static AI image generators

**Cost Impact:** +$0.25 per video (TTS + Lip Sync)

---

### Phase 3: Compression & Delivery (v3.2) 📝 DESIGNED

**Objective:** Professional-quality delivery with manageable file sizes

**Design Document:** `docs-design-Compression_Delivery_Pipeline_v3.2-v1.0-2026_01_10.md`

**Deliverables:**
| Component | Description | Status |
|-----------|-------------|--------|
| Cloudinary Compression | q_auto:eco, f_auto:video, c_limit,w_1080 | 📝 Designed |
| Binary File Delivery | Actual file attachment, not URL | 📝 Designed |
| Dual Filename Convention | Internal (archive) + External (user-facing) | 📝 Designed |
| Google Drive Archive | Comprehensive logging with metadata | 📝 Designed |
| Telegram Display Fix | Explicit width/height for proper rendering | 📝 Designed |

**New Nodes:** 8-10

**Filename Conventions:**

| Type | Format | Example |
|------|--------|---------|
| **Internal (Archive)** | `PwP_{method}_{date}_{time}_{execId}.mp4` | `PwP_telegram_2026-01-10_14-32-05_abc123.mp4` |
| **External (User)** | `Picture_with_Picasso-AtelierBenito-{execId}-{date}.mp4` | `Picture_with_Picasso-AtelierBenito-abc123-2026_01_10.mp4` |

**What It Achieves:**
- File size reduced from 20-30 MB to 5-10 MB
- Users receive actual video file (works offline)
- Complete audit trail in Google Drive
- Fixes Telegram mobile display distortion

**Benefit:**
- Fast downloads on any connection
- Professional file naming
- Easy to find/reference any video
- Reliable display across devices

**Cost Impact:** +$0.01 per video (Cloudinary)

---

### Phase 4: Enhanced Delivery 🎯 TARGET

**Objective:** Brand-ready, social-media-friendly output

**Deliverables:**
| Component | Description | Status |
|-----------|-------------|--------|
| Branded Message | Inspirational intro with attribution | 🎯 To Design |
| Hashtag Caption | Platform-ready hashtags | 🎯 To Design |
| Content Formatting | Optimized for sharing | 🎯 To Design |

**New Nodes:** 2-3

**Message Template:**
```
✨ Your Picture with Picasso moment has arrived!

🎨 An Atelier Benito joint, produced by RobotBird.AI

#AtelierBenito #PictureWithPicasso #AIArt #Picasso
#GenerativeArt #RobotBirdAI #DigitalArt #AIVideo
```

**What It Achieves:**
- Every video delivered with brand attribution
- Hashtags ready for copy/paste to social
- Professional, consistent messaging

**Benefit:**
- Brand recognition with every delivery
- Users can share immediately without editing
- Consistent brand voice across all outputs
- Organic marketing through user shares

**Cost Impact:** None (text formatting only)

---

## Phase 1-4 Completion Summary

**Total New Nodes (Phase 2-4):** ~22 nodes

**File Size Journey:**
| Stage | Size |
|-------|------|
| Kling O1 Output (Silent) | 18-25 MB |
| After Audio (Phase 2) | 20-30 MB |
| After Compression (Phase 3) | 5-10 MB |

**Cost Per Video (Phase 4 Complete):**
| Component | Cost |
|-----------|------|
| Kling O1 | $1.00 |
| ElevenLabs TTS | $0.05 |
| LatentSync | $0.20 |
| Cloudinary | $0.01 |
| **Total** | **~$1.26** |

---

## Future Enhancements: Phases 5-9

### Phase 5: Social Posting Integration 📋 FUTURE

**Objective:** One-click posting to social media platforms

**Prerequisites:** Phase 4 complete, API approvals obtained

**Deliverables:**
| Component | Description | Priority |
|-----------|-------------|----------|
| Video Watermark | "An Atelier Benito Joint Created by RobotBird.AI" | High |
| Telegram Buttons | Inline keyboard for platform selection | High |
| Instagram Reels | Post via Instagram Graph API | High |
| TikTok | Post via Content Posting API | High |
| X (Twitter) | Post via X API v2 | Medium |
| Facebook | Post via Facebook Graph API | Medium |

**Social Platform Details:**

| Platform | API | Video Support | Auth Requirement | Complexity |
|----------|-----|---------------|------------------|------------|
| Instagram | Instagram Graph API | Reels | Facebook Business Account | Medium |
| TikTok | Content Posting API | Native | TikTok Developer App | Medium |
| X | X API v2 | Native | X Developer Account | Low |
| Facebook | Facebook Graph API | Feed + Reels | Facebook Business Account | Low |

**Button Flow:**
```
[Video Delivered to User]
        │
        ▼
[Message with 4 buttons]
├── 📸 Post to Instagram
├── 🎵 Post to TikTok
├── 𝕏 Post to X
└── 📘 Post to Facebook
        │
        ▼
[User taps button]
        │
        ▼
[n8n posts to selected platform]
        │
        ▼
[Confirmation sent to user]
```

**Watermark Specification:**
| Element | Value |
|---------|-------|
| Text | "An Atelier Benito Joint Created by RobotBird.AI" |
| Position | Bottom center or bottom right |
| Style | Semi-transparent, professional font |
| Implementation | Cloudinary text overlay or FFmpeg burn-in |

**What It Achieves:**
- Users can share directly from delivery message
- Every shared video carries brand attribution (watermark)
- Reduces friction from "receive video" to "post video"

**Benefit:**
- Viral marketing potential
- Brand exposure on every social share
- User convenience drives more sharing
- Organic reach without ad spend

**Estimated Development:** 2-3 weeks
**API Approval Timeline:** 2-4 weeks (TikTok, Instagram)

---

### Phase 6: Web Frontend 📋 FUTURE

**Objective:** Professional web presence on RobotBird.io

**Prerequisites:** Phase 5 complete (or parallel development)

**Deliverables:**
| Component | Description |
|-----------|-------------|
| Landing Page | RobotBird.io/picture-with-picasso |
| Photo Upload | Drag-and-drop or file picker |
| Progress Tracking | Real-time status updates |
| Video Preview | Watch before download/share |
| Download Button | Direct file download |
| Share Buttons | Social posting (connects to Phase 5) |

**Technology Stack:**
| Layer | Technology |
|-------|------------|
| Framework | Next.js 14+ |
| Hosting | Vercel |
| Styling | Tailwind CSS |
| State | React Query |
| Backend Connection | n8n webhooks |

**What It Achieves:**
- Accessible without Telegram/Email
- Professional brand experience
- Broader audience reach

**Benefit:**
- Not limited to messaging app users
- SEO-discoverable
- Portfolio piece for RobotBird.AI
- Foundation for SaaS conversion

**Estimated Development:** 3-4 weeks

---

### Phase 7: SaaS Foundation 📋 FUTURE

**Objective:** Monetizable product with user accounts

**Prerequisites:** Phase 6 complete

**Deliverables:**
| Component | Description |
|-----------|-------------|
| User Authentication | Sign up, login, account management |
| Stripe Integration | Payment processing |
| Subscription Tiers | Free, Creator, Pro, Enterprise |
| Usage Tracking | Credits, limits, history |
| Admin Dashboard | User management, analytics |

**Pricing Model (Draft):**
| Tier | Price | Credits | Features |
|------|-------|---------|----------|
| Free | $0 | 1/month | Watermarked, no social posting |
| Creator | $9.99/mo | 10/month | No watermark, social posting |
| Pro | $29.99/mo | 50/month | Priority processing, analytics |
| Enterprise | Custom | Unlimited | API access, white-label |

**Technology Stack:**
| Component | Technology |
|------------|------------|
| Authentication | Clerk or Supabase Auth |
| Payments | Stripe |
| Database | Supabase (Postgres) |
| Backend | n8n + Supabase Functions |

**What It Achieves:**
- Revenue generation
- User accounts with history
- Scalable business model

**Benefit:**
- Sustainable business
- User retention through accounts
- Upsell opportunities
- Data for product improvement

**Estimated Development:** 4-6 weeks

---

### Phase 8: Mobile & Messaging Expansion 📋 FUTURE

**Objective:** Maximum accessibility across platforms

**Prerequisites:** Phase 7 complete (or partial)

**Deliverables:**
| Component | Description | Priority |
|-----------|-------------|----------|
| WhatsApp Integration | Input via WhatsApp Business API | High |
| iOS App | Native or React Native | Medium |
| Android App | Native or React Native | Medium |
| PWA | Progressive Web App | Medium |
| Discord Bot | Community integration | Low |
| SMS Input | Twilio integration | Low |

**Mobile App Features:**
- Camera integration (take photo directly)
- Photo library picker
- Push notifications for completion
- Video gallery of past creations
- Share sheet integration

**What It Achieves:**
- Reach users on their preferred platform
- Native mobile experience
- Push notification engagement

**Benefit:**
- Larger addressable market
- Better user experience
- Higher engagement through notifications
- Competitive with native apps

**Estimated Development:** 6-8 weeks (React Native for both platforms)

---

### Phase 9: Scale & Enterprise 📋 FUTURE

**Objective:** High-volume, enterprise-ready platform

**Prerequisites:** Phase 8 complete, significant user base

**Deliverables:**
| Component | Description |
|-----------|-------------|
| Multi-Tenant Architecture | Isolated environments per enterprise client |
| Queue Management | Handle high-volume concurrent requests |
| Analytics Dashboard | Usage metrics, performance data |
| White-Label Option | Remove RobotBird branding for enterprise |
| API Access | Programmatic integration for partners |
| SLA Guarantees | Uptime, processing time commitments |

**What It Achieves:**
- Enterprise sales capability
- High-volume processing
- B2B revenue stream

**Benefit:**
- Premium pricing for enterprise
- Recurring B2B revenue
- Platform for multiple AI experiences
- Foundation for additional "Picture with [Artist]" products

---

## Technical Architecture Overview

### Current Architecture (Phase 1-4)

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT CHANNELS                        │
│                                                         │
│    [Telegram Bot]              [Email]                  │
│         │                         │                     │
└─────────┴─────────────────────────┴─────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                   n8n WORKFLOW                          │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Route &   │  │  Kling O1   │  │   Audio     │     │
│  │   Validate  │─▶│   Video     │─▶│  Pipeline   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                           │             │
│                                           ▼             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Delivery   │◀─│   Format    │◀─│ Cloudinary  │     │
│  │   Route     │  │   Message   │  │  Compress   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │                                               │
└─────────┴───────────────────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
    [Telegram]      [Email]    [Google Drive]
```

### Future Architecture (Phase 5-9)

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT CHANNELS                        │
│                                                         │
│ [Telegram] [Email] [Web] [WhatsApp] [iOS] [Android]    │
│      │        │      │       │        │        │        │
└──────┴────────┴──────┴───────┴────────┴────────┴───────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              AUTHENTICATION & BILLING                    │
│                                                         │
│         [Clerk/Supabase Auth]  [Stripe]                 │
│                    │               │                    │
└────────────────────┴───────────────┴────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   n8n WORKFLOW                          │
│         (Same core + social posting nodes)              │
└─────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
┌─────────────────┐ ┌─────────────┐ ┌─────────────────┐
│ Direct Delivery │ │   Social    │ │    Archive      │
│ Telegram/Email/ │ │   Posting   │ │  Google Drive + │
│ Web/App         │ │ IG/TT/X/FB  │ │  Supabase       │
└─────────────────┘ └─────────────┘ └─────────────────┘
```

---

## External Service Dependencies

### Current (Phase 1-4)

| Service | Purpose | Required |
|---------|---------|----------|
| fal.ai | Kling O1, ElevenLabs, LatentSync | Yes |
| Cloudinary | Video compression | Yes |
| Telegram | Input/Output channel | Yes |
| Gmail | Input/Output channel | Yes |
| Google Drive | Archive storage | Yes |

### Future (Phase 5-9)

| Service | Purpose | Phase |
|---------|---------|-------|
| Instagram Graph API | Social posting | 5 |
| TikTok API | Social posting | 5 |
| X API | Social posting | 5 |
| Facebook Graph API | Social posting | 5 |
| Vercel | Web hosting | 6 |
| Clerk/Supabase | Authentication | 7 |
| Stripe | Payments | 7 |
| WhatsApp Business API | Messaging | 8 |
| Apple App Store | iOS distribution | 8 |
| Google Play Store | Android distribution | 8 |

---

## Risk Assessment

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| fal.ai API changes | High | Low | Monitor changelog, abstract API calls |
| Social API approval delays | Medium | Medium | Apply early, have backup channels |
| Cost overruns at scale | Medium | Medium | Usage limits, tier pricing |
| Telegram display bug persists | Low | High | Explicit dimensions, user education |
| Competition launches similar | Medium | Medium | Brand differentiation, speed to market |

---

## Success Metrics

### Phase 4 Success Criteria

| Metric | Target |
|--------|--------|
| Video file size | 5-10 MB |
| End-to-end processing time | < 5 minutes |
| Delivery success rate | > 95% |
| User satisfaction (qualitative) | Positive feedback |

### Future Success Metrics

| Metric | Target | Phase |
|--------|--------|-------|
| Social shares per video | > 0.3 (30% share rate) | 5 |
| Web conversion rate | > 5% visitors to users | 6 |
| Paid conversion rate | > 2% free to paid | 7 |
| Monthly recurring revenue | $1,000+ | 7 |
| Mobile app rating | > 4.5 stars | 8 |

---

## Related Documents

| Document | Purpose | Location |
|----------|---------|----------|
| Audio Pipeline Design | Phase 2 technical spec | `docs/design/docs-design-Audio_Pipeline_v3.1-v1.0-2026_01_10.md` |
| Compression Pipeline Design | Phase 3 technical spec | `docs/design/docs-design-Compression_Delivery_Pipeline_v3.2-v1.0-2026_01_10.md` |
| **Data Dictionary** | Data elements, Supabase schema, metrics | `docs/design/docs-design-Data_Dictionary-v1.0-2026_01_10.md` |
| **Metrics Tracker** | Excel spreadsheet for manual tracking | `docs/tracking/PwP_Metrics_Tracker-v1.0-2026_01_10.xlsx` |
| BRD | Original business requirements | `docs/design/PICTURE-WITH-PICASSO-BRD.md` |
| Workflow Plan | Workflow architecture | `docs/design/PICTURE-WITH-PICASSO-WORKFLOW-PLAN.md` |
| Kling O1 Config | API configuration | `docs/design/design-kling-o1-configuration-v1.0-2026-01-06.md` |

---

## Revision History

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2026-01-10 | 1.1 | Added Data Dictionary and Metrics Tracker | Claude Code |
| 2026-01-10 | 1.0 | Initial roadmap document | Claude Code |

---

## Approval

| Role | Name | Date | Status |
|------|------|------|--------|
| Project Owner | Atelier Benito | | Pending |
| Technical Lead | | | Pending |

---

*Document created: 2026-01-10*
*Last updated: 2026-01-10*
