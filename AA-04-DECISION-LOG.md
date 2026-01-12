# Decision Log

**Project:** Picture with Picasso
**Last Updated:** 2026-01-12

---

## 2026-01-12

### Decision: Initialize GSD Project Management System

**Category:** Process / Project Management
**Decision:** Adopt GSD (Get Shit Done) hierarchical planning system for the Interactive Audio Pipeline
**Rationale:**
- Provides structured approach for 10-phase implementation
- Session continuity via STATE.md
- Allows YOLO mode for faster iteration
**Alternatives Considered:**
- Continue ad-hoc planning (rejected - too complex for 10 phases)
- Use traditional project management tools (rejected - not integrated with Claude Code)
**Impact:** All future work tracked via .planning/ directory
**Implemented:** Yes - PROJECT.md, ROADMAP.md, STATE.md, config.json created

---

### Decision: YOLO Mode + Comprehensive Depth

**Category:** Process / Configuration
**Decision:** Use YOLO mode (skip all confirmation gates) with Comprehensive depth (detailed analysis)
**Rationale:**
- User prefers autonomous execution
- Comprehensive ensures thorough work despite fewer checkpoints
**Alternatives Considered:**
- Normal mode (rejected - too many confirmations)
- Express depth (rejected - insufficient detail for complex APIs)
**Impact:** Faster iteration, detailed plans without interruptions
**Implemented:** Yes - config.json configured

---

### Decision: ElevenLabs Direct API (Not via fal.ai)

**Category:** Technical / API Integration
**Decision:** Access ElevenLabs TTS and Sound Effects APIs directly, not through fal.ai wrapper
**Rationale:**
- Full control over voice parameters
- No wrapper limitations
- Direct access to Voice Design and Voice Cloning APIs
**Alternatives Considered:**
- fal.ai wrapper (rejected - may not expose all features)
**Impact:** More flexibility, requires direct HTTP Request nodes in n8n
**Reversible:** Yes - could switch to wrapper if needed
**Implemented:** Pending (Phase 5)

---

### Decision: Input Channels - Telegram + Web Form Only

**Category:** Technical / Architecture
**Decision:** Support only Telegram bot and web form input channels (no email)
**Rationale:**
- User confirmed these are the only active channels
- Simplifies input parsing
- Reduces maintenance burden
**Alternatives Considered:**
- Add email channel (rejected - not currently used)
**Impact:** Two input paths to maintain, not three
**Implemented:** Pending (Phase 1)

---

### Decision: Visual Quality as Core Value

**Category:** Product / Quality
**Decision:** Visual quality and identity preservation is THE core value - not audio
**Rationale:**
- User quote: "What destroys the magic is quirky interaction of the figures and the people not looking like themselves"
- Picasso must be proportional (5'4" / 163cm)
- Face morphing/stretching destroys likeness
**Alternatives Considered:**
- Audio quality as core value (rejected - video quality more critical)
**Impact:** All decisions should prioritize likeness preservation
**Implemented:** Yes - documented in PROJECT.md

---

### Decision: Dual Project Structure

**Category:** Process / Organization
**Decision:** Use BOTH GSD's `.planning/` directory AND existing project structure (source/, docs/, versions/)
**Rationale:**
- GSD provides planning rigor and session continuity
- Existing structure contains production artifacts
- Complementary purposes, no conflict
**Alternatives Considered:**
- Replace existing structure with GSD (rejected - existing structure has value)
- Skip GSD (rejected - need planning for 10 phases)
**Impact:** Two organizational systems coexist
**Implemented:** Yes

---

## 2026-01-10

### Decision: Use Polling Design for Audio Pipeline (Not Webhooks)

**Category:** Technical / Architecture
**Decision:** Implement 11-node polling design instead of 4-node webhook design for TTS + lip sync
**Rationale:**
- Polling has 95% confidence vs 30-65% for webhooks
- Matches existing Kling polling pattern (proven, 9 nodes)
- Single execution context eliminates metadata passing issues
- Can optimize to webhooks later without rework
**Alternatives Considered:**
- Webhook design (4-6 nodes) - rejected due to:
  - `fal_webhook` must be query param, not body (critical)
  - Metadata passing between executions is fragile
  - No fallback mechanism for webhook failures
  - `$workflow.url` unverified in n8n
**Impact:** +11 nodes to workflow, +30-60s latency per video, but reliable
**Reversible:** Yes - can migrate to webhooks after production stability
**Implemented:** Design approved, awaiting implementation

---

### Decision: Document fal.ai Webhook Capability for Future Use

**Category:** Technical / Research
**Decision:** Documented that fal.ai supports webhooks via `fal_webhook` query parameter
**Rationale:**
- Discovered during research: `https://queue.fal.run/fal-ai/{model}?fal_webhook={url}`
- Could reduce polling loops from 9 nodes to 2 nodes (78% reduction)
- But implementation has significant complexity
**Impact:** Informs future optimization decisions
**Status:** Documented in `docs/analysis/docs-analysis-Audio_Pipeline_Architecture_Analysis-v1.0-2026_01_10.md`
**Implemented:** Research complete

---

## 2026-01-09

### Decision: Kling O1 reference-to-video Does Not Support Audio

**Category:** Technical / API Investigation
**Decision:** Confirmed that `fal-ai/kling-video/o1/reference-to-video` endpoint does NOT support `generate_audio` parameter
**Rationale:**
- Scraped fal.ai API documentation - `generate_audio` not in input schema
- Parameter is silently ignored when sent
- Audio generation only exists on Kling 2.6 endpoints
**Evidence:**
- Workflow Q2Z6nJYPotQnhlwj includes `generate_audio: true` - confirmed via n8n MCP
- Execution 3984 produced video without audio despite parameter
- API docs show no `generate_audio` for reference-to-video endpoint
**Impact:** Major - need alternative approach for audio capability
**Status:** Confirmed - this is an API limitation, not a bug

---

### Decision: Explore 2-Step Pipeline Architecture

**Category:** Technical / Architecture Research
**Decision:** Research image composition → Kling 2.6 as potential solution
**Rationale:**
- Kling 2.6 I2V has `generate_audio: true` and it works
- But Kling 2.6 has no `elements` array for multi-identity
- Composition step could combine identities first
**Options Researched:**
- Reve Remix (`fal-ai/reve/remix`) - 1-6 images
- FLUX.2 Pro Edit - up to 9 images
- FLUX.1 Kontext Max Multi - experimental
**Trade-off:** Risk of Picasso identity loss during composition
**Status:** Superseded by TTS + lip sync approach (2026-01-10)

---

### Decision: Prompt v1.5 Character Limit Fix

**Category:** Technical / Prompt Engineering
**Decision:** Reduce prompt from ~5,600 to ~2,150 characters (under 2,500 API limit)
**Rationale:**
- Execution 3981 failed: `422 - ensure this value has at most 2500 characters`
- Consolidated verbose sections, moved prohibitions to negative prompt
**What Was Removed:**
- Entire DO NOT section (moved to negative prompt)
- Verbose if/then structures
- Separate PICASSO/VISITORS subsections
**What Was Preserved:**
- Core narrative, @Element2 as visual master, identity preservation
- Audio generation instructions (even though API ignores them)
- All technical specifications
**Impact:** Prompt now fits API limit
**Implemented:** Yes - v1.5 created

---

## 2026-01-08 (Evening)

### Decision: Negative Prompt Content for Kling O1

**Category:** Technical / Quality Control
**Decision:** Implement 8-item negative prompt for identity preservation and quality control
**Content:**
```
face morphing, identity swap, duplicated limbs, extra limbs, distorted anatomy, unnatural proportions, changing background, motion blur
```
**Rationale:**
- Identity items (face morphing, identity swap) - Kling O1 best practices for multi-element videos
- Anatomical items (duplicated limbs, extra limbs, distorted anatomy) - Proven controls from v2.4 Reve Remix
- Proportions (unnatural proportions) - Maintains realistic sizing per v2.4 5'4" reference
- Environment (changing background) - User requirement for scene consistency
- Quality (motion blur) - Video quality control from v2.4
**Sources:**
- v2.4 Reve Remix prompt quality controls
- Kling O1 documentation best practices
- User requirements
**Impact:** Better identity preservation and quality control in generated videos
**Implemented:** Yes - added to node JSON v1.1

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

*Updated: 2026-01-12*
