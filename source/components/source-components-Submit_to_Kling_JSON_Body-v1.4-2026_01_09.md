# Submit to Kling1 - JSON Body Configuration

**Version:** v1.4-2026-01-09
**Node:** Submit to Kling1
**Field:** jsonBody (under Send Body → JSON)

---

## JSON Body Expression

Copy this entire expression into the `jsonBody` field:

```javascript
={{ JSON.stringify({
  "prompt": $json.prompt,
  "negative_prompt": $json.negative_prompt,
  "elements": $json.elements,
  "duration": $json.duration,
  "aspect_ratio": $json.aspect_ratio,
  "generate_audio": true
}) }}
```

---

## What Changed from v1.3

| Parameter | v1.3 | v1.4 |
|-----------|------|------|
| `generate_audio` | Missing | `true` |

**This is the root cause of missing audio in v1.3 tests.**

---

## Full API Request Structure

When the expression evaluates, it produces this JSON:

```json
{
  "prompt": "[v1.4 prompt text from Prepare Kling Elements1]",
  "negative_prompt": "[v1.4 negative prompt from Prepare Kling Elements1]",
  "elements": [
    {
      "reference_image_urls": ["data:image/...;base64,..."],
      "frontal_image_url": "data:image/...;base64,..."
    },
    {
      "reference_image_urls": ["data:image/...;base64,..."],
      "frontal_image_url": "data:image/...;base64,..."
    }
  ],
  "duration": "10",
  "aspect_ratio": "9:16",
  "generate_audio": true
}
```

---

## Node Location

**Workflow:** `Picture with Picasso -v.3-v2.4` (ID: Q2Z6nJYPotQnhlwj)
**Node:** Submit to Kling1
**Path in UI:** Click node → Parameters → Send Body → Specify Body → JSON

---

## How to Update

1. Open workflow in n8n
2. Click on `Submit to Kling1` node
3. Scroll to **Send Body** section
4. Find **JSON** field
5. Replace entire contents with the expression above
6. Save workflow

---

## Verification

After updating, the node parameters should show:

| Field | Value |
|-------|-------|
| Method | POST |
| URL | `https://queue.fal.run/fal-ai/kling-video/o1/reference-to-video` |
| Authentication | Generic Credential Type (Header Auth) |
| Send Body | On |
| Specify Body | Using JSON |
| JSON | `={{ JSON.stringify({...}) }}` with `generate_audio: true` |

---

## Related Files

| File | Purpose |
|------|---------|
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.4-2026_01_09.md` | Prompt and negative prompt text |
| `docs/changelogs/docs-changelogs-Changelog_v2.4_to_v3.0-v1.0-2026_01_08.md` | Full change documentation |
