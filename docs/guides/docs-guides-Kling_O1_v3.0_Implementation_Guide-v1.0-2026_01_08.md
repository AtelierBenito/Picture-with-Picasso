# Kling O1 v3.0 Implementation Guide

**Version:** v1.0-2026-01-08
**Audience:** Workflow administrators performing manual node insertion
**Estimated Time:** 30-45 minutes

---

## Overview

This guide provides step-by-step instructions for manually inserting the Kling O1 nodes into the Picture with Picasso v2.4 workflow to create v3.0.

---

## Prerequisites

Before starting, ensure you have:

- [ ] Access to n8n cloud instance with workflow ID `C22rlgmTijUZsUSb`
- [ ] fal.ai account with Kling O1 API access
- [ ] FAL_KEY API key ready
- [ ] Backup of current v2.4 workflow

---

## Step 1: Backup Current Workflow

1. Open n8n and navigate to Picture with Picasso workflow
2. Click **...** menu → **Download**
3. Save as `Picture with Picasso -Wan 2.5-v2.4-BACKUP-{date}.json`
4. Copy to `versions/workflows/` in project directory

---

## Step 2: Identify Nodes to Remove

Remove the following 13 nodes from the workflow:

### Reve Remix Pipeline (6 nodes)

| # | Node Name | How to Find |
|---|-----------|-------------|
| 1 | **Merge Images for Reve** | Connected to both Base64 conversion nodes |
| 2 | **Send to Reve Remix** | HTTP POST to fal.ai reve endpoint |
| 3 | **Check Reve Status** | HTTP GET for reve status |
| 4 | **Check Reve Complete** | If node checking reve status |
| 5 | **Wait for Reve** | Wait node in reve polling loop |
| 6 | **Download Composed Image** | HTTP GET for composed image |

### Cloudinary + Kie.ai Pipeline (7 nodes)

| # | Node Name | How to Find |
|---|-----------|-------------|
| 7 | **Upload Composed Image to Cloudinary** | HTTP POST to Cloudinary |
| 8 | **Archive Composed Image to Drive** | Google Drive node (optional removal) |
| 9 | **Prepare Video Prompt** | Set node before video request |
| 10 | **Create Video Request** | HTTP POST to Kie.ai |
| 11 | **Get Videos** | HTTP GET for Kie.ai status |
| 12 | **If** (video status check) | If node checking video status |
| 13 | **Wait** (video polling) | Wait node in video polling loop |

### How to Remove Nodes

1. Select each node
2. Press **Delete** or right-click → **Delete**
3. Connections will be automatically removed

---

## Step 3: Open Node Insert JSON

Open the file:
```
source/workflows/source-workflows-Kling_O1_Nodes_Insert-v1.0-2026_01_08.json
```

This file contains:
- 10 new nodes in the `nodes` array
- Internal connections in the `connections` object
- Documentation of insertion/exit points

---

## Step 4: Add New Nodes

### Option A: Copy-Paste Individual Nodes

For each node in the JSON `nodes` array:

1. In n8n, click **+** to add a node
2. Search for the node type (e.g., "Set", "HTTP Request")
3. Configure parameters to match the JSON configuration
4. Position nodes logically in the canvas

### Option B: Import Partial JSON (Advanced)

1. Create a temporary workflow with just the new nodes
2. Copy nodes from temporary to main workflow
3. Delete temporary workflow

### Node Positions (Suggested)

Arrange nodes in a logical flow:

```
                    ┌──────────────────────────┐
                    │  Prepare Kling Elements  │
                    └────────────┬─────────────┘
                                 ↓
                    ┌──────────────────────────┐
                    │    Submit to Kling       │
                    └────────────┬─────────────┘
                                 ↓
                    ┌──────────────────────────┐
                    │ Initialize Retry Counter │
                    └────────────┬─────────────┘
                                 ↓
    ┌───────────────────────────────────────────────┐
    │              Check Kling Status               │←──────────────┐
    └───────────────────────┬───────────────────────┘               │
                            ↓                                        │
    ┌───────────────────────────────────────────────┐               │
    │               Status Router                   │               │
    └──┬────────────────┬───────────────────────┬───┘               │
       ↓                ↓                       ↓                    │
  COMPLETED          FAILED              PENDING (x2)               │
       ↓                ↓                       ↓                    │
┌──────────────┐  ┌──────────┐   ┌───────────────────────┐          │
│Get Kling     │  │ Error    │   │ Increment Retry       │          │
│Result        │  │ Handling │   │ Counter               │          │
└──────┬───────┘  └──────────┘   └───────────┬───────────┘          │
       ↓                                      ↓                      │
┌──────────────┐              ┌───────────────────────┐             │
│Download      │              │   Check Max Retries   │             │
│Kling Video   │              └───────────┬───────────┘             │
└──────┬───────┘                   TRUE   │   FALSE                 │
       ↓                            ↓     │     ↓                   │
   [Exit to                    ┌────────┐ │ ┌────────────────┐      │
   Archive &                   │Timeout │ │ │Wait for Kling  │──────┘
   Route Output]               │Error   │ │ │(20 seconds)    │
                               └────────┘ │ └────────────────┘
```

---

## Step 5: Connect Insertion Points

### From Existing Nodes to New Nodes

| From Node | To Node | Connection Type |
|-----------|---------|-----------------|
| **Convert User Image to Base64** | **Prepare Kling Elements** | Main output → Input 0 |
| **Convert Picasso Image to Base64** | **Prepare Kling Elements** | Main output → Input 1 |

### Steps:
1. Find `Convert User Image to Base64` node
2. Drag connection from its output
3. Connect to `Prepare Kling Elements` input
4. Repeat for `Convert Picasso Image to Base64`

---

## Step 6: Connect Internal Nodes

Connect the new nodes in sequence:

| From Node | To Node | Output |
|-----------|---------|--------|
| Prepare Kling Elements | Submit to Kling | Main |
| Submit to Kling | Initialize Retry Counter | Main |
| Initialize Retry Counter | Check Kling Status | Main |
| Check Kling Status | Status Router | Main |
| Status Router | Get Kling Result | Output 0 (COMPLETED) |
| Status Router | Increment Retry Counter | Output 1 (FAILED) |
| Status Router | Increment Retry Counter | Output 2 (PENDING) |
| Status Router | Increment Retry Counter | Output 3 (PENDING) |
| Get Kling Result | Download Kling Video | Main |
| Increment Retry Counter | Check Max Retries | Main |
| Check Max Retries | Wait for Kling | Output 1 (FALSE) |
| Wait for Kling | Check Kling Status | Main (creates loop) |

---

## Step 7: Connect Exit Points

### From New Nodes to Existing Nodes

| From Node | To Node | Purpose |
|-----------|---------|---------|
| **Download Kling Video** | **Archive Video to Google Drive** | Save video to Drive |
| **Download Kling Video** | **Route Output** | Deliver to Telegram/Email |

### Steps:
1. Find `Download Kling Video` node
2. Drag connection to `Archive Video to Google Drive`
3. Drag another connection to `Route Output`

---

## Step 8: Configure Error Handling

The `Check Max Retries` TRUE output needs error handling:

### Option A: Connect to Existing Error Flow
If workflow has existing error notification nodes, connect there.

### Option B: Add Simple Error Logging
1. Add a **Set** node after Check Max Retries TRUE output
2. Configure to set error message:
   ```
   errorType: "TIMEOUT"
   errorMessage: "Video generation timed out after 10 minutes"
   request_id: {{ $json.request_id }}
   ```
3. Connect to a notification node (Telegram/Email)

---

## Step 9: Verify API Key Configuration

Ensure the FAL_KEY is accessible:

### Option A: Workflow Configuration Node
1. Find the `Workflow Configuration` node
2. Add field: `falApiKey`
3. Enter your fal.ai API key

### Option B: Environment Variable
1. In n8n Settings → Variables
2. Add: `FAL_KEY` = your fal.ai API key
3. Update HTTP nodes to use: `{{ $env.FAL_KEY }}`

---

## Step 10: Update Workflow Metadata

1. Click on workflow name
2. Rename to: `Picture with Picasso v3.0`
3. Update any description/notes

---

## Step 11: Test the Workflow

### Test 1: Manual Trigger Test
1. Prepare test images (user photo + Picasso image)
2. Trigger workflow manually
3. Monitor execution flow
4. Verify video output

### Test 2: Verify Polling Loop
1. Watch the polling loop execute
2. Confirm status checks every 20 seconds
3. Verify completion detection

### Test 3: Verify Output Routing
1. Test Telegram delivery path
2. Test Email delivery path
3. Verify Google Drive archival

---

## Troubleshooting

### Issue: HTTP 401 Unauthorized
**Cause:** Invalid FAL_KEY
**Solution:** Verify API key in Workflow Configuration or environment variables

### Issue: Polling Loop Never Completes
**Cause:** Status never reaches COMPLETED
**Solution:** Check fal.ai dashboard for request status, verify request_id is being passed correctly

### Issue: Video Not Downloading
**Cause:** video.url expression incorrect
**Solution:** Verify Get Kling Result returns `video.url` field, check expression syntax

### Issue: Connections Not Working
**Cause:** Data not flowing between nodes
**Solution:** Use n8n execution view to inspect data at each step

---

## Validation Checklist

Before activating:

- [ ] All 13 old nodes removed
- [ ] All 10 new nodes added
- [ ] Insertion points connected (2 connections)
- [ ] Exit points connected (2 connections)
- [ ] Internal connections complete (polling loop works)
- [ ] FAL_KEY configured
- [ ] Error handling configured
- [ ] Manual test successful
- [ ] Video output verified

---

## Rollback Procedure

If issues arise:

1. Deactivate v3.0 workflow
2. Import backup from `versions/workflows/`
3. Rename imported workflow
4. Activate restored v2.4 workflow
5. Investigate v3.0 issues

---

## Files Reference

| File | Purpose |
|------|---------|
| `source/workflows/source-workflows-Kling_O1_Nodes_Insert-v1.0-2026_01_08.json` | Node configurations to insert |
| `source/prompts/source-prompts-Kling_O1_Video_Prompt-v1.0-2026_01_08.md` | Video generation prompt |
| `source/components/source-components-Kling_O1_v3.0_Configuration-v1.0-2026_01_08.md` | Complete configuration reference |
| `docs/changelogs/docs-changelogs-Changelog_v2.4_to_v3.0-v1.0-2026_01_08.md` | Change documentation |
| `docs/research/docs-research-Kling_O1_Background_Analysis-v1.0-2026_01_08.md` | Background preservation research |

---

## Support

If you encounter issues not covered in this guide:

1. Check fal.ai API status: https://status.fal.ai
2. Review n8n execution logs for detailed error messages
3. Consult the configuration document for parameter details

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-08 | v1.0 | Initial implementation guide |
