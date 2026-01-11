# Design: Lessons Learned Documentation Pattern

**Version:** v1.0-2026-01-10
**Purpose:** Define a context-efficient pattern for documenting lessons learned
**Target:** Skills Management and Creating Project (and all future projects)

---

## Problem Statement

When lessons learned are loaded into AI assistant context:
- Verbose documents consume excessive tokens
- Key insights get buried in investigation details
- No quick way to scan for relevant learnings

---

## Solution: Three-Tier Documentation

Separate documentation into three tiers optimized for different use cases:

```
┌─────────────────────────────────────────────────────────────────┐
│  TIER 1: README Index (docs/lessons-learned/README.md)          │
│  ─────────────────────────────────────────────────────────────  │
│  • One-line summaries in table format                           │
│  • Links to Tier 2 documents                                    │
│  • Purpose: Quick scanning for humans, navigation for Claude    │
│  • No size limit - grows with project                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  TIER 2: Lessons Learned (docs/lessons-learned/*.md)            │
│  ─────────────────────────────────────────────────────────────  │
│  • Key takeaways and actionable guidance                        │
│  • References Tier 3 for full details                           │
│  • Purpose: Learning, prevention checklist                      │
│  • Succinct by default, verbose when the lesson requires it     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  TIER 3: Issue Document (docs/Bug Tracker/issues/resolved/*.md) │
│  ─────────────────────────────────────────────────────────────  │
│  • Full investigation notes                                     │
│  • Step-by-step reproduction                                    │
│  • Technical deep dives                                         │
│  • Purpose: Debugging similar issues, auditing                  │
│  • No size limit - complete record                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tier Specifications

### Tier 1: README Index

**Location:** `docs/lessons-learned/README.md`

**Purpose:** Quick-scan index for all lessons learned

**Format:**
```markdown
# Lessons Learned Index

| ID | Title | Category | Date | Summary |
|----|-------|----------|------|---------|
| [LL-001](file.md) | Short Title | Category | YYYY-MM-DD | One sentence summary |
```

**Rules:**
- One row per lesson
- Summary ≤ 15 words
- Links to Tier 2 document
- Sorted by date (newest first) or by category

---

### Tier 2: Lessons Learned Document

**Location:** `docs/lessons-learned/docs-lessons-learned-[Topic]-v[X.Y]-[YYYY_MM_DD].md`

**Purpose:** Concise, actionable guidance

**Template:**
```markdown
# Lessons Learned: [Topic]

**ID:** LL-XXX
**Date:** YYYY-MM-DD
**Category:** [Category]
**Related Issue:** [Link to Tier 3]

---

## Summary

[2-3 sentences describing the problem and solution]

---

## Key Lessons

### 1. [First Lesson]
[1-2 sentences]

### 2. [Second Lesson]
[1-2 sentences]

---

## Quick Reference

[Table, formula, or checklist for immediate use]

---

## Full Details

See: [Link to Issue Document]

---

*Last Updated: YYYY-MM-DD*
```

**Rules:**
- Succinct by default - avoid unnecessary verbosity
- Verbose when necessary - never sacrifice facts or lessons to save space
- Focus on WHAT to do, not investigation history
- Include actionable quick reference (formula, checklist, table)
- Always link to Tier 3 for full investigation details

---

### Tier 3: Issue Document

**Location:** `docs/Bug Tracker/issues/resolved/docs-Bug_Tracker-issues-resolved-ISSUE_XXX_[Topic]-v[X.Y]-[YYYY_MM_DD].md`

**Purpose:** Complete investigation record

**Template:** Use standard issue template from `docs/Bug Tracker/templates/ISSUE-TEMPLATE.md`

**Rules:**
- No length limit
- Include all investigation steps
- Technical deep dives welcome
- Link back to Tier 2 lessons learned

---

## Cross-Reference Pattern

```
README Index (Tier 1)
    └── links to → Lessons Learned (Tier 2)
                       └── links to → Issue Document (Tier 3)

Issue Document (Tier 3)
    └── links to → Lessons Learned (Tier 2)
```

Both Tier 2 and Tier 3 reference each other for bidirectional navigation.

---

## Context Loading Strategy

| Use Case | Load |
|----------|------|
| Quick check for relevant learnings | Tier 1 only |
| Apply known solution | Tier 1 + specific Tier 2 |
| Debug similar issue | Tier 1 + Tier 2 + Tier 3 |

**Default recommendation:** Load Tier 1 (README index) at session start. Load Tier 2/3 only when relevant to the task at hand.

---

## Folder Structure

```
docs/
    lessons-learned/
        README.md                           # Tier 1: Index
        docs-lessons-learned-[Topic]-v1.0-YYYY_MM_DD.md  # Tier 2

    Bug Tracker/
        issues/
            open/                           # Active issues
            resolved/                       # Tier 3: Full details
                docs-Bug_Tracker-issues-resolved-ISSUE_XXX_[Topic]-v1.0-YYYY_MM_DD.md
        templates/
            ISSUE-TEMPLATE.md
```

---

## Naming Convention

| Tier | Pattern |
|------|---------|
| Tier 1 | `README.md` (fixed name) |
| Tier 2 | `docs-lessons-learned-[Topic]-v[X.Y]-[YYYY_MM_DD].md` |
| Tier 3 | `docs-Bug_Tracker-issues-resolved-ISSUE_XXX_[Topic]-v[X.Y]-[YYYY_MM_DD].md` |

---

## ID System

| Type | Format | Example |
|------|--------|---------|
| Lessons Learned | LL-XXX | LL-001, LL-002 |
| Issues | ISSUE-XXX | ISSUE-001, ISSUE-002 |

IDs are project-scoped (not global).

---

## Benefits

1. **Context efficiency:** Load only what's needed
2. **Quick scanning:** README index for fast lookup
3. **Actionable guidance:** Tier 2 focused on solutions
4. **Complete record:** Tier 3 preserves full investigation
5. **Bidirectional links:** Easy navigation between tiers

---

## Implementation Checklist

For new projects adopting this pattern:

- [ ] Create `docs/lessons-learned/README.md` with index table
- [ ] Create `docs/Bug Tracker/issues/resolved/` folder
- [ ] Copy issue template to `docs/Bug Tracker/templates/`
- [ ] Document pattern in project CLAUDE.md

---

## Example Implementation

See Picture with Picasso project:
- Tier 1: `docs/lessons-learned/README.md`
- Tier 2: `docs/lessons-learned/docs-lessons-learned-API_Character_Limits-v1.0-2026_01_10.md`
- Tier 3: `docs/Bug Tracker/issues/resolved/docs-Bug_Tracker-issues-resolved-ISSUE_001_Prompt_Character_Limit-v1.0-2026_01_10.md`

---

*Last Updated: 2026-01-10*
