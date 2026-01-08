# Picture with Picasso Workflow Redesign

## Overview
Complete workflow restructure to fix expression errors, eliminate Cloudinary dependency, consolidate archive nodes, and implement correct Reve Remix integration.

---

## Documentation Sources

| Source | URL | Used For |
|--------|-----|----------|
| fal.ai Reve Remix API | https://fal.ai/models/fal-ai/reve/remix/api | API schema validation |
| fal.ai Performance Guide | https://fal.ai/learn/devs/gen-ai-performance-optimization | Large file handling |
| n8n-mcp | `nodes-base.editImage` | Image resize node |
| n8n Official Docs | https://docs.n8n.io/code/cookbook/expressions/common-issues/ | Expression error fix |

---

## Current vs. New Architecture

### CURRENT (Broken)
```
TELEGRAM: Trigger → Download → Upload to Cloudinary → Prepare Input → Get Picasso
FORM:     Trigger → Workflow Config → Download → Archive Form Image → Get Picasso

Problems:
- Expression error (no path to Workflow Configuration)
- Cloudinary dependency
- Two separate archive nodes
- Incorrect Reve JSON structure
```

### NEW (Unified)
```
TELEGRAM: Trigger ─┬─→ Workflow Configuration ─┬─→ Edit Image → Convert Base64 ─┐
                   │                           │                                 │
FORM:     Trigger ─┘                           └─→ Archive User Image (PARALLEL) │
                                                                                 │
                   Get Picasso Images → Select Random → Download → Convert Base64│
                                                                                 │
                                               ┌─────────────────────────────────┘
                                               ▼
                                    Merge Images → Send to Reve Remix
```

---

## Changes Summary

### NODES TO DELETE (4 nodes)

| Node Name | Line | Reason |
|-----------|------|--------|
| Upload Telegram Photo to Cloudinary | ~895 | Replaced by base64 |
| Archive Telegram Image to Drive | ~1010 | Consolidated |
| Archive Form Image to Drive | ~976 | Consolidated |
| Prepare Telegram Input | ~493 | Merged into unified flow |

### NODES TO ADD (5 new nodes)

| Node Name | Type | Purpose |
|-----------|------|---------|
| Edit Image (Resize) | `n8n-nodes-base.editImage` | Resize images to max 2048px |
| Convert User Image to Base64 | `n8n-nodes-base.code` | Convert binary to base64 string |
| Convert Picasso Image to Base64 | `n8n-nodes-base.code` | Convert binary to base64 string |
| Merge Images for Reve | `n8n-nodes-base.merge` | Combine both images |
| Send to Reve Remix | `n8n-nodes-base.httpRequest` | Submit to fal.ai API |
| Archive User Image | `n8n-nodes-base.googleDrive` | Single shared archive node |

### NODES TO MODIFY (3 nodes)

| Node Name | Change |
|-----------|--------|
| Workflow Configuration | Remove Cloudinary fields, add Reve API key |
| Telegram Trigger | Connect to Workflow Configuration |
| Download Telegram Photo | Connect to Edit Image |

### CONNECTIONS TO CHANGE

**Remove:**
- `Telegram Trigger → Download Telegram Photo` (direct)
- `Download Telegram Photo → Upload Telegram Photo to Cloudinary`
- `Form Trigger → Workflow Configuration` (keep but add Telegram path)

**Add:**
- `Telegram Trigger → Workflow Configuration`
- `Form Trigger → Workflow Configuration`
- `Workflow Configuration → Edit Image (Resize)` (main path)
- `Workflow Configuration → Archive User Image` (parallel path)
- `Edit Image → Convert User Image to Base64`
- `Download Picasso Image → Convert Picasso Image to Base64`
- `Convert User Base64 + Convert Picasso Base64 → Merge Images for Reve`
- `Merge Images for Reve → Send to Reve Remix`

---

## Detailed Node Configurations

### 1. Edit Image (Resize)
```json
{
  "operation": "resize",
  "width": 2048,
  "height": 2048,
  "resizeOption": "contain",
  "dataPropertyName": "data"
}
```

### 2. Convert User Image to Base64 (Code Node)
```javascript
const binaryData = $input.first().binary.data;
const base64String = binaryData.data;
const mimeType = binaryData.mimeType;
return [{
  json: {
    userImageBase64: `data:${mimeType};base64,${base64String}`
  }
}];
```

### 3. Convert Picasso Image to Base64 (Code Node)
```javascript
const binaryData = $input.first().binary.data;
const base64String = binaryData.data;
const mimeType = binaryData.mimeType;
return [{
  json: {
    picassoImageBase64: `data:${mimeType};base64,${base64String}`
  }
}];
```

### 4. Merge Images for Reve
- Type: Merge
- Mode: Combine
- Combines userImageBase64 and picassoImageBase64 into single item

### 5. Send to Reve Remix (HTTP Request)
```json
{
  "method": "POST",
  "url": "https://fal.run/fal-ai/reve/remix",
  "authentication": "genericCredentialType",
  "headers": {
    "Authorization": "Key {{ $env.FAL_KEY }}",
    "Content-Type": "application/json"
  },
  "body": {
    "prompt": "USER WILL ENTER PROMPT HERE",
    "image_urls": [
      "{{ $json.userImageBase64 }}",
      "{{ $json.picassoImageBase64 }}"
    ],
    "aspect_ratio": "16:9",
    "num_images": 1,
    "output_format": "png"
  }
}
```

### 6. Archive User Image (Google Drive)
- Operation: Upload
- Folder: `{{ $('Workflow Configuration').first().json.userUploadedImagesFolderId }}`
- Binary Property: data

---

## Workflow Configuration Updates

**REMOVE these fields:**
- cloudinaryCloudName
- cloudinaryApiKey

**KEEP these fields:**
- duration
- resolution
- optimizePrompt
- videoArchiveFolderId
- userUploadedImagesFolderId

**ADD this field:**
- falApiKey (or use n8n credentials)

---

## Reve Remix Prompt Location

**Node:** Send to Reve Remix
**Field:** `body.prompt`
**User will manually enter the prompt in this node after implementation**

Example prompt format:
```
"Place the person from <img>0</img> into an artistic scene inspired by <img>1</img>,
matching the painting style while keeping the person realistic."
```

Note: `<img>0</img>` = user image, `<img>1</img>` = Picasso image

---

## File to Modify
- `Picture with Picasso -Wan 2.5.json`

---

## Validation Checklist

Before implementation:
- [ ] Both triggers route to Workflow Configuration
- [ ] Single Archive node (parallel, non-blocking)
- [ ] Cloudinary nodes removed
- [ ] Edit Image node resizes to max 2048px
- [ ] Base64 conversion for both images
- [ ] Correct Reve Remix JSON structure (no invalid fields)
- [ ] Prompt field left empty for user input
- [ ] All existing node configurations preserved where applicable
