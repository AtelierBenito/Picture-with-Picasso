# Lessons Learned: API Character Limits & JSON Encoding Overhead

**ID:** LL-001
**Date:** 2026-01-10
**Category:** API Integration
**Related Issue:** [ISSUE-001](../Bug%20Tracker/issues/resolved/docs-Bug_Tracker-issues-resolved-ISSUE_001_Prompt_Character_Limit-v1.0-2026_01_10.md)

---

## Summary

API prompts that appear under the character limit can fail after JSON encoding adds overhead. In n8n multi-node workflows, double-escaping can add 7-10% to character count.

---

## The Problem

| Prompt v1.7 | Raw Chars | After JSON | Limit | Result |
|-------------|-----------|------------|-------|--------|
| Under limit | 2,350 | 2,538 | 2,500 | **FAILED** |

---

## Key Lessons

### 1. Keep 10% Below Limit

For Kling O1 prompt (2,500 limit): **target 2,250 max**

### 2. JSON Adds Hidden Overhead

In n8n workflows with `JSON.stringify()`, special characters get double-escaped:
- Each `"` adds ~2 chars
- Each `\n` adds ~2 chars
- Total overhead: 7-10%

### 3. Test Actual Payload

Check n8n execution logs → HTTP Request node → Request Body for true character count.

### 4. Use `negative_prompt` for Prohibitions

Move "DO NOT" directives to `negative_prompt` field to save space in main prompt.

---

## API Limits for This Workflow

| API | Field | Limit |
|-----|-------|-------|
| Kling O1 | `prompt` | 2,500 chars |
| Kling O1 | `negative_prompt` | Undocumented (~1,500 safe) |
| ElevenLabs TTS | `text` | 10,000+ chars (not a constraint) |

---

## Quick Reference

```
API Limit × 0.90 = Safe Target

Kling O1: 2,500 × 0.90 = 2,250 max
```

---

## Full Details

See: [ISSUE-001 - Prompt Character Limit](../Bug%20Tracker/issues/resolved/docs-Bug_Tracker-issues-resolved-ISSUE_001_Prompt_Character_Limit-v1.0-2026_01_10.md)

---

*Last Updated: 2026-01-10*
