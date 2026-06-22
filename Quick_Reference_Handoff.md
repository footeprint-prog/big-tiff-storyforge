# Big Tiff StoryForge — Quick Reference: Multi-Agent Handoff

**Last Updated:** June 17, 2026

## File Locations

| File | Location | Owner |
|------|----------|-------|
| `Big_Tiff_Scene_Structure.json` | GitHub: https://github.com/footeprint-prog/big-tiff-storyforge.git | Sammy (updates), Dom (reads) |
| Master Story Bible + Reference Docs | Google Drive (Big Tiff folder) | Sammy |
| Webtool files (`writing.html`, etc.) | This Grok Project + `/home/workdir/artifacts/Big-Tiff-StoryForge/` | Dom |

## Agent Roles (Quick)

**Webcraft Master Dom** (Primary in this Project)
- Builds and maintains the webtool
- Reads `Big_Tiff_Scene_Structure.json` from GitHub
- Default agent when working on the tool

**StoryForge Sammy**
- Maintains story canon and `Big_Tiff_Scene_Structure.json` in GitHub
- Updates scene structure, prompts, and statuses

**Grok** (Project Orchestrator)
- Strategic oversight and coordination
- Use when you want high-level architecture or cross-agent alignment

## Key Rules

- **Never** auto-edit or delete Erica’s drafts.
- `review` status requires **intentional user action** to change.
- Major structural changes must be discussed with Aaron first.
- All scene structure changes go through GitHub (`Big_Tiff_Scene_Structure.json`).

## How to Trigger a Full Refresh

Say: **“Refresh yourself”**

This will cause a full review of:
- Project vision and principles
- Checklist and changelog
- Current state of the webtool
- Alignment between agents

## Quick Commands

| Goal | What to Say |
|------|-------------|
| Work on the tool | (Just start talking — Dom responds by default) |
| Strategic / architecture input | “Switch to Grok” or “Think as Project Orchestrator” |
| Full alignment check | “Refresh yourself” |
| Update scene structure | Tell Sammy to update the JSON in GitHub |

---

**Keep this file handy for quick reference during development.**