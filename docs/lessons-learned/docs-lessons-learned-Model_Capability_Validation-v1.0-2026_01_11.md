# Lesson Learned: Validate Model Capabilities Before Tool Change Recommendations

**ID:** LL-002
**Date:** 2026-01-11
**Category:** Tool Selection

---

## Summary

Never recommend switching video generation tools based on changelog announcements alone. Validate that the new model supports ALL workflow requirements before suggesting a change.

---

## What Happened

WAN 2.6 was surfaced as a potential improvement after reviewing Kie.ai changelog. Upon deeper validation, WAN 2.6 **cannot** support the core workflow requirement:

- **Requirement:** Combine 2 images (visitor photo + Picasso environment)
- **WAN 2.6 Reality:** Image-to-Video accepts only 1 image; Reference-to-Video requires VIDEO inputs

---

## Key Validation Checklist

Before recommending a model change, verify:

| Requirement | Must Validate |
|-------------|---------------|
| Multi-image composition | Does it accept 2+ image inputs? |
| Identity preservation | Can it maintain face/likeness? |
| Prompt syntax | How are multiple subjects referenced? |
| Prompt limits | Is limit sufficient for current prompts? |
| API availability | Is it available via fal.ai/n8n? |

---

## Root Cause

Changelog research showed new models but did not validate against specific workflow requirements before surfacing as recommendations.

---

## Prevention

1. **Changelogs show features, not limitations** - always check API docs
2. **Test against THE workflow** - not general capabilities
3. **Validate before recommending** - research depth before tool change suggestions

---

*Related: Kling O1 Elements system (`@Element1`, `@Element2`) remains the validated approach for 2-image composition.*
