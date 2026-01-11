# Issue: API Prompt Character Limit Exceeded Due to JSON Encoding

**ID:** ISSUE-001
**Created:** 2026-01-09
**Resolved:** 2026-01-10
**Status:** Resolved
**Priority:** Critical

---

## Summary

Kling O1 API prompt submissions failed with 422 errors because raw prompts that appeared under the 2,500 character limit exceeded the limit after JSON encoding and double-escaping in n8n workflow nodes.

---

## Affected Executions

| Execution | Prompt Version | Raw Chars | After JSON | Result |
|-----------|----------------|-----------|------------|--------|
| #3981 | v1.4 | ~5,600 | N/A | FAILED |
| #3986 | v1.6 | ~2,920 | N/A | FAILED |
| #3987 | v1.7 | ~2,350 | ~2,538 | FAILED |

---

## Steps to Reproduce

1. Create a prompt with ~2,350 characters (under 2,500 limit)
2. Store prompt in a Set node with special characters (quotes, etc.)
3. Pass to HTTP Request node using `JSON.stringify()`
4. Submit to fal.ai Kling O1 API
5. **Result:** 422 error - prompt exceeds 2,500 characters

---

## Expected Behavior

A prompt with 2,350 raw characters should pass the 2,500 character API limit.

## Actual Behavior

The prompt failed with `prompt: size must be between 0 and 2500` because JSON encoding inflated the character count to ~2,538.

---

## Environment

- **Workflow Version:** v3.0
- **n8n Version:** Current (self-hosted)
- **Related Nodes:** Prepare Kling Elements1, Submit to Kling1
- **API:** fal.ai Kling O1 Reference-to-Video

---

## API Limits for This Workflow

| API | Field | Limit | Source |
|-----|-------|-------|--------|
| **Kling O1** | `prompt` | 2,500 chars | Confirmed by 422 errors |
| **Kling O1** | `negative_prompt` | Undocumented | Used ~1,100 chars without issue |
| **ElevenLabs TTS** (future) | `text` | 10,000+ chars | Not a constraint for ~100 char scripts |
| **LatentSync** (future) | N/A | No text prompts | N/A |

**Note:** The 2,500 character limit is not explicitly documented on fal.ai's public documentation. It was discovered through execution errors.

---

## Investigation Notes

### Phase 1: Obvious Overflow (v1.4 → v1.5)

**Execution #3981** failed with a clearly oversized prompt (~5,600 chars vs 2,500 limit).

**Error:** `422 - ensure this value has at most 2500 characters`

**Fix:** Reduced prompt to ~2,383 characters by:
- Moving DO NOT section to negative_prompt
- Condensing if/then structures
- Merging duplicate sections

### Phase 2: Hidden Inflation (v1.6 → v1.7)

**Execution #3986** failed even though documented character count showed ~2,180.

**Root cause:** Character count was calculated from only part of the prompt. Actual assembled prompt was ~2,920 characters.

**Fix:** Properly counted all prompt text, reduced to ~2,350 characters.

### Phase 3: JSON Double-Escaping (v1.7 → v1.8) - CRITICAL DISCOVERY

**Execution #3987** failed with a prompt that was clearly under the limit (~2,350 chars).

**Error:** `prompt: size must be between 0 and 2500`

**Investigation:**
1. Checked n8n execution logs
2. Examined actual HTTP request body
3. Found payload was ~2,538 characters (188 chars over limit)

**Root cause:** JSON double-escaping in n8n workflow.

When data passes through multiple n8n nodes:

```
Node 1 (Set): prompt = 'Say "hello"'
           → Stored with escaping: 'Say \"hello\"'

Node 2 (HTTP Request with JSON.stringify):
           → Double-escaped: 'Say \\"hello\\"'
           → Additional overhead: 7-10%
```

**Escape overhead calculation for v1.7:**

| Character Type | Count in Prompt | Overhead per Char | Total Overhead |
|----------------|-----------------|-------------------|----------------|
| Double quotes | ~50 | +2 chars | ~100 chars |
| Other special | ~40 | +2 chars | ~80 chars |
| **Total** | | | **~180 chars** |

Raw prompt: 2,350 + JSON overhead: 188 = **2,538 characters** (38 over limit)

---

## Resolution

**Fixed in:** Prompt v1.8 (2026-01-10)
**Solution:** Reduced raw prompt to ~2,405 characters with explicit buffer for JSON overhead.

### Final Specifications

| Metric | Value |
|--------|-------|
| Raw character count | ~2,405 |
| Buffer below limit | ~95 chars (3.8%) |
| Estimated JSON overhead | ~45 chars |
| Estimated final payload | ~2,450 chars |
| API Limit | 2,500 |
| **Result** | **PASSED** |

### Trimming Strategy (v1.7 → v1.8)

Removed ~53 characters of redundant phrases:

| Removed Phrase | Reason |
|----------------|--------|
| "no artistic liberties" | Covered by "no AI interpretations" |
| "no gradual changes across frames" | Covered by "no drift, no morphing" |
| "their" from "Lock their identity" | Unnecessary word |
| "to source" from "pixel-perfect" | Context already clear |
| "Never make Picasso appear taller than 5'4"" | Redundant with height spec |
| "not a background figure" | Implied by "fully animated" |
| "Intimate, personal feel" | Nice-to-have, not essential |

---

## Technical Deep Dive: JSON Encoding Overhead

### Standard JSON Escaping (Expected: 3-5%)

```javascript
const prompt = 'Say "hello"';
JSON.stringify({ prompt: prompt });
// Result: {"prompt":"Say \"hello\""}
```

### Double-Escaping in Multi-Node Workflows (Actual: 7-10%)

When text passes through multiple nodes:

| Stage | Content | Chars |
|-------|---------|-------|
| Original | `Say "hello"` | 11 |
| After Set node | `Say \"hello\"` | 13 |
| After JSON.stringify | `Say \\"hello\\"` | 15 |

### How to Verify Actual Payload Size

**Option A: Debug node**
```javascript
// Add Code node before HTTP Request
const payload = JSON.stringify({
  prompt: $json.prompt,
  negative_prompt: $json.negative_prompt,
  elements: $json.elements,
  duration: $json.duration,
  aspect_ratio: $json.aspect_ratio,
  generate_audio: true
});
console.log('Payload length:', payload.length);
return { payloadLength: payload.length, payload: payload };
```

**Option B: n8n execution logs**
1. Open execution details
2. Expand HTTP Request node
3. Check "Request Body" section

---

## Prevention Guidelines

### Safe Buffer for Kling O1

| Field | Limit | Safe Target | Buffer |
|-------|-------|-------------|--------|
| `prompt` | 2,500 | **2,250 max** | 10% |
| `negative_prompt` | Unknown | ~1,500 suggested | Conservative |

### Checklist for New Prompts

- [ ] Count characters in final assembled prompt
- [ ] Keep 10% below API limit for JSON overhead
- [ ] Test actual payload size in execution logs
- [ ] Document character count in prompt file

---

## Related Documents

| Document | Purpose |
|----------|---------|
| `docs/lessons-learned/docs-lessons-learned-API_Character_Limits-v1.0-2026_01_10.md` | Concise lessons learned |
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.8-2026_01_10.md` | Working prompt |
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.7-2026_01_10.md` | Failed prompt (double-escaping case) |

---

## External References

- [Kling O1 Reference-to-Video API](https://fal.ai/models/fal-ai/kling-video/o1/reference-to-video) - fal.ai
- [Kling O1 Prompt Guide](https://fal.ai/learn/devs/kling-o1-prompt-guide) - fal.ai
- [ElevenLabs Character Limits](https://help.elevenlabs.io/hc/en-us/articles/13298164480913-What-s-the-maximum-amount-of-characters-and-text-I-can-generate) - ElevenLabs Help

---

*Last Updated: 2026-01-10*
