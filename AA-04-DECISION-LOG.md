# Decision Log

**Project:** Picture with Picasso
**Last Updated:** 2026-01-08

---

## 2026-01-08

### Decision: Migrate to Kling O1 Reference-to-Video (v3.0)

**Category:** Technical / API Stack
**Decision:** Replace Reve Remix + Kie.ai with Kling O1 Reference-to-Video (fal.ai) for direct multi-identity video generation
**Rationale:**
- Single-stage architecture eliminates intermediate image composition step
- Native @Element system preserves both user and Picasso identities
- Cost reduction (one API call instead of two)
- Latency improvement (faster end-to-end processing)
- Unified platform on fal.ai
**Alternatives Considered:**
- Keep Reve Remix + Kie.ai (rejected - two API calls, more complex)
- WAN 2.6 (rejected - reference tracking limitations)
**Impact:** Major architectural change, requires manual node insertion
**Reversible:** Yes - v2.4 workflow preserved as backup
**Implemented:** Artifacts created, ready for manual insertion

---

### Decision: 9:16 Vertical Format for Social Media

**Category:** Technical / Output Format
**Decision:** Change aspect ratio from 16:9 (horizontal) to 9:16 (vertical)
**Rationale:**
- Native format for TikTok, Instagram Reels, Stories
- YouTube Shorts support
- Social media ready without reformatting
**Alternatives Considered:**
- Keep 16:9 (rejected - requires reformatting for social)
- 1:1 square (rejected - not optimal for any platform)
**Impact:** Video output immediately shareable on social platforms
**Reversible:** Yes - aspect_ratio parameter change
**Implemented:** Yes - configured in v3.0 nodes

---

### Decision: Descriptive Prompt Format (Not Procedural)

**Category:** Technical / Prompt Engineering
**Decision:** Use descriptive prompts (what to show) instead of procedural (how to process)
**Rationale:**
- Kling O1 generates video, doesn't compose images
- Visual cues should come from @Element2 reference image
- Removes hardcoded environment descriptions
**Alternatives Considered:**
- Port v2.4 procedural instructions (rejected - wrong format for video generation)
**Impact:** Cleaner prompts focused on creative intent
**Reversible:** Yes - prompt can be modified
**Implemented:** Yes - finalized in source/prompts/

---

### Decision: Manual Node Insertion (Not Full Workflow Regeneration)

**Category:** Technical / Implementation Strategy
**Decision:** Create standalone JSON for manual node insertion rather than regenerating entire workflow
**Rationale:**
- Preserve working v2.4 workflow during migration
- Reduce risk of breaking existing functionality
- Allow incremental validation
- Avoid over-taxing AI processing on complex 1450-line workflow
**Alternatives Considered:**
- Regenerate full workflow (rejected - too risky, workflow is complex)
**Impact:** Safer migration path, requires manual work in n8n
**Reversible:** Yes - can restore v2.4 from backup
**Implemented:** Yes - JSON file created with documentation

---

## 2025-11-30

### Decision: "Route BEFORE Download" Architecture Pattern

**Category:** Technical / Workflow Architecture
**Decision:** Tag input source explicitly at trigger level using Set nodes, then route, then download - instead of routing after download
**Rationale:**
- Download nodes (Telegram, HTTP) don't preserve original trigger JSON data
- Previous approach checked `$json.message` AFTER download, but that data was lost
- Explicit `inputSource` field set at trigger level guarantees reliable routing
**Alternatives Considered:**
- Infer source from data after download (rejected - data not preserved)
- Pass source through binary property name (rejected - unreliable)
**Impact:** Reliable routing for both Telegram and Form inputs
**Reversible:** Yes - connection/node changes only
**Implemented:** Yes - v2.1 JSON created

---

### Decision: Use Base64 Data URIs for User Images

**Category:** Technical / API Integration
**Decision:** Convert user images to base64 data URI format (`data:image/...;base64,...`) for fal.ai Reve Remix API
**Rationale:**
- Telegram provides only `file_id`, not permanent public URLs
- fal.ai Reve Remix accepts data URIs in `image_urls` array
- Eliminates need for temporary URL hosting (Cloudinary removed)
- Unified handling for both Telegram and Form inputs
**Alternatives Considered:**
- Cloudinary temporary URLs (rejected - added dependency, no longer needed)
- S3 presigned URLs (rejected - adds infrastructure complexity)
**Impact:** Simpler architecture, no external image hosting dependency
**Reversible:** Yes - could add URL hosting later if needed
**Implemented:** Yes - v2.1 JSON includes base64 conversion nodes

---

### Decision: Single Unified Archive Node

**Category:** Technical / Workflow Simplification
**Decision:** Consolidate "Archive Telegram Image to Drive" and "Archive Form Image to Drive" into single "Archive User Image" node
**Rationale:**
- Both archive to same Google Drive folder
- Same operation regardless of source
- Reduces node count and maintenance
**Alternatives Considered:**
- Keep separate archive nodes (rejected - unnecessary duplication)
**Impact:** Cleaner workflow, single point of configuration
**Reversible:** Yes - could split back to separate nodes
**Implemented:** Yes - v2.1 uses single Archive node (needs v2.2 fix for connection)

---

### Decision: Eliminate Cloudinary Dependency (for user image upload)

**Category:** Technical / Infrastructure Simplification
**Decision:** Remove Cloudinary upload for user images, use base64 data URIs instead
**Rationale:**
- Cloudinary was used to host user images temporarily for API access
- fal.ai accepts base64 data URIs directly
- Reduces external service dependencies
- Simplifies credential management
**Alternatives Considered:**
- Keep Cloudinary for URL generation (rejected - base64 is simpler)
**Impact:** One less external service to configure and maintain
**Reversible:** Yes - could add back if URL hosting needed
**Implemented:** Yes - Cloudinary nodes removed in v2

---

## 2025-11-27

### Decision: Switch to Kie.ai (Wan 2.5) for Video Generation

**Category:** Technical / Platform Selection
**Decision:** Use Kie.ai API with Wan 2.5 model for image-to-video generation, replacing previous face-swap approach
**Rationale:**
- Proven working template exists (Skool community)
- Simple 2-endpoint API pattern (createTask, recordInfo)
- English documentation, straightforward onboarding
- Fair pricing ($0.05-0.10/second)
- User's "keep it simple" requirement
**Alternatives Considered:**
- Alibaba DashScope direct API (rejected - unnecessary complexity for minimal cost savings)
- Replicate face-swap + video (rejected - user prefers simpler approach)
- Pollo AI (rejected - also an aggregator, no advantage over Kie.ai)
**Impact:** Simpler implementation, faster time to production
**Reversible:** Yes - HTTP Request nodes can point to different APIs
**Implemented:** Design complete, pending credential configuration

---

### Decision: Single Unified Workflow (Not 3 Separate Workflows)

**Category:** Technical / Architecture
**Decision:** Use one workflow with dual triggers (Form + Telegram) and dual outputs (Email + Telegram), with source-based routing
**Rationale:**
- User explicitly requested unified workflow
- n8n supports normalize-and-consolidate pattern
- Single source of truth, less maintenance
- No code duplication between input channels
**Alternatives Considered:**
- 3 separate workflows (rejected - user doesn't want this)
- Separate trigger workflows calling shared workflow (rejected - adds complexity)
**Impact:** Cleaner architecture, easier maintenance
**Reversible:** Yes - could split into separate workflows later
**Implemented:** Design complete in `Picture with Picasso -Wan 2.5.json`

---

### Decision: Archive Previous Design Documents

**Category:** Documentation / Project Management
**Decision:** Move `PICTURE-WITH-PICASSO-WORKFLOW-DESIGN.md` and `WORKFLOW-VALIDATION-REPORT.md` to archive folder
**Rationale:**
- User provided new workflow design (JSON file)
- Old design used different approach (face-swap + Replicate)
- Keep project root clean with current design only
- Historical documents preserved in archive
**Alternatives Considered:**
- Delete old files (rejected - lose historical context)
- Keep both in root (rejected - confusing which is current)
**Impact:** Clear project structure, current design is obvious
**Reversible:** Yes - files in archive/
**Implemented:** Yes

---

## 2025-11-26

### Decision: Use Face Swap Instead of AI Image Generation

**Category:** Technical / Platform Selection
**Decision:** Composite users into REAL historical Picasso photographs using face-swap technology
**Status:** SUPERSEDED by 2025-11-27 decision to use Wan 2.5 image-to-video
**Note:** Original approach archived, replaced with simpler video generation

---

### Decision: Replicate as Primary AI Platform

**Category:** Technical / Infrastructure
**Decision:** Use Replicate for both image compositing and video generation
**Status:** SUPERSEDED by 2025-11-27 decision to use Kie.ai
**Note:** Kie.ai chosen for simplicity

---

### Decision: ElevenLabs for Voice Generation

**Category:** Technical / Audio
**Decision:** Use ElevenLabs API with Spanish-accented historical voice
**Status:** ON HOLD - Current workflow doesn't include voice generation
**Note:** May revisit for future enhancement

---

### Decision: Telegram/WhatsApp as Primary Input Method

**Category:** UX / Integration
**Decision:** Support Telegram and webhook/form triggers as primary input methods
**Status:** CURRENT - Implemented in new workflow design
**Update:** WhatsApp dropped in favor of web Form Trigger for simplicity

---

### Decision: Optional Animation Branch Architecture

**Category:** Technical / Workflow Design
**Decision:** Implement animation as optional branch
**Status:** SUPERSEDED - New workflow generates video by default (no image-only option)

---

## Decision Template

```markdown
### Decision: [Title]

**Category:** [Process/Technical/Documentation/UX/etc.]
**Decision:** [What was decided]
**Rationale:**
- [Why this decision]
- [What problem it solves]
**Alternatives Considered:**
- [Option 1] (rejected - reason)
- [Option 2] (rejected - reason)
**Impact:** [Effect on project]
**Reversible:** [Yes/No - how to reverse if needed]
**Implemented:** [Yes / Pending / Design phase]
```

---

*Updated: 2025-11-27*
