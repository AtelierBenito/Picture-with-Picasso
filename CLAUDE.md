# Picture with Picasso - Project Configuration

<!-- structure-version: 1.7 -->

**Last Updated:** 2026-01-08

## Project Overview

n8n workflow project for generating "Picture with Picasso" images using Wan 2.5 and Reve AI integration.

## Project Structure

```
Picture with Picasso/
    CLAUDE.md                           # Project configuration (this file)

    source/                             # ACTIVE ARTIFACTS
        workflows/                      # Active workflow JSON files
        prompts/                        # Agent prompts
        components/                     # Configs, forms, integrations
        guides/                         # Production-critical docs
        media/
            images/                     # Image files
            videos/                     # Video files
            automation/                 # Screenshots, recordings

    docs/                               # DOCUMENTATION
        changelogs/                     # Version history
        lessons-learned/                # Lessons learned log
        analysis/                       # RCA, recovery plans
        design/                         # Architecture docs
        guides/                         # Setup, config guides
        research/                       # Background research
        schemas/                        # Data structures
        templates/                      # Reusable patterns
        Bug Tracker/                    # Issue and problem tracking
            issues/open/                # Active issues
            issues/resolved/            # Fixed issues
            problems/active/            # Active problems
            problems/resolved/          # Solved problems
            templates/                  # Issue/problem templates

    versions/                           # HISTORICAL
        ARCHIVE-LOG.md                  # Archive index
        sessions/                       # Old session summaries
        workflows/                      # Old workflow versions
        prompts/                        # Old prompt versions
        schemas/                        # Old schema versions

    Picasso Images/                     # Reference image collection

    AA-01-WHERE-WE-ARE-NOW.md           # Session dashboard
    AA-02-SESSION-SUMMARY-LATEST.md     # Latest session summary
    AA-03-NEXT-STEPS.md                 # Next steps
    AA-04-DECISION-LOG.md               # Decision log
    AA-05-DOCUMENT-INVENTORY.md         # Document inventory
```

## Quick Navigation

| Need | Location |
|------|----------|
| Current workflow | `source/workflows/` |
| Old workflow versions | `versions/workflows/` |
| Design documents | `docs/design/` |
| Configuration guides | `docs/guides/` |
| Picasso reference images | `Picasso Images/` |
| Session status | `AA-01-WHERE-WE-ARE-NOW.md` |

## Key Files

- **Active Workflow:** `source/workflows/Picture with Picasso -Wan 2.5-v2.4.json`
- **BRD:** `docs/design/PICTURE-WITH-PICASSO-BRD.md`
- **Workflow Plan:** `docs/design/PICTURE-WITH-PICASSO-WORKFLOW-PLAN.md`

## Development Standards

- Workflow versions use format: `*-vX.Y.json`
- Latest version goes in `source/workflows/`
- Previous versions archived to `versions/workflows/`
- Session files (AA-*) remain at project root

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-08 | 1.1 | Added Bug Tracker structure via project-setup |
| 2026-01-06 | 1.0 | Initial CLAUDE.md created via project-migrate |
