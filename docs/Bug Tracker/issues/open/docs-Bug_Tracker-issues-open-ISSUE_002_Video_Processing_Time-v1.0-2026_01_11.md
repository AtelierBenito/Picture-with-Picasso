# ISSUE-002: Video Processing Time Optimization

**Status:** Open
**Priority:** Medium
**Created:** 2026-01-11
**Category:** Performance

---

## Problem Statement

Current workflow takes **6 minutes** to complete. This is too slow for interactive use cases and will compound when Phase 2 audio pipeline is added.

---

## Root Cause

95%+ of processing time is spent in Kling O1 video generation:

| Phase | Time | % of Total |
|-------|------|------------|
| Image download/resize | ~5-10 sec | ~2% |
| **Kling O1 video generation** | **~5-6 min** | **~95%** |
| Output (archive/send) | ~5-10 sec | ~2% |

---

## Current Configuration

| Parameter | Value | Location |
|-----------|-------|----------|
| Video duration | 10 seconds | `Prepare Kling Elements1` node |
| Resolution | 720p | `Workflow Configuration` node |
| Audio flag | `generate_audio: true` | `Submit to Kling1` node |
| Polling interval | 20 seconds | `Wait for Kling1` node |
| Max retries | 30 | `Check Max Retries1` node |
| Endpoint | O1 (reference-to-video) | `Submit to Kling1` node |

---

## Recommended Optimizations

### Option 1: Reduce Video Duration (HIGH IMPACT)

**Change:** 10 seconds → 5 seconds

**Expected Result:** ~50% time reduction (6 min → 3 min)

**Trade-off:** Less video content per generation

**Implementation:**
```
Node: Prepare Kling Elements1
Field: duration
Current: "10"
New: "5"
```

---

### Option 2: Remove Audio Flag (LOW IMPACT)

**Change:** Remove `generate_audio: true` from API call

**Expected Result:** ~10-30 seconds saved

**Trade-off:** None (Kling O1 reference-to-video doesn't support audio anyway)

**Implementation:**
```
Node: Submit to Kling1
Field: jsonBody
Remove: "generate_audio": true
```

---

### Option 3: Reduce Polling Interval (LOW IMPACT)

**Change:** 20 seconds → 12 seconds

**Expected Result:** ~10-20 seconds saved (faster completion detection)

**Trade-off:** Slightly more API calls (negligible cost)

**Implementation:**
```
Node: Wait for Kling1
Field: amount
Current: 20
New: 12
```

---

### Option 4: Test Standard Kling Endpoint (UNKNOWN IMPACT)

**Change:** Switch from O1 to standard Kling

**Expected Result:** Unknown - needs testing

**Trade-off:** Potentially lower quality output

**Implementation:**
```
Node: Submit to Kling1
Current URL: https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video
Test URL: https://queue.fal.run/fal-ai/kling-video/v1/reference-to-video
```

---

## Combined Optimization Estimate

| Optimization | Time Saved |
|--------------|------------|
| 5-second duration | ~3 min |
| Remove audio flag | ~10-30 sec |
| 12-second polling | ~10-20 sec |
| **Total** | **~3-3.5 min** |

**Result:** 6 min → ~2.5-3 min

---

## Acceptance Criteria

- [ ] Workflow completes in under 3 minutes
- [ ] Video quality remains acceptable
- [ ] No increase in failure rate

---

## Files Affected

- `source/workflows/Picture with Picasso -v.3-v2.4.json`

---

## Notes

- Phase 2 audio pipeline will add additional processing time (ElevenLabs TTS + LatentSync lip sync)
- Optimize v3.0 before adding Phase 2 complexity
- Consider user expectations: 3 min may still feel slow for interactive use

---

## Related Documents

- Core Requirements Checklist: `docs/design/docs-design-Core_Requirements_Checklist-v1.0-2026_01_11.md`
- Workflow: `source/workflows/Picture with Picasso -v.3-v2.4.json`

---

*Issue created: 2026-01-11*
