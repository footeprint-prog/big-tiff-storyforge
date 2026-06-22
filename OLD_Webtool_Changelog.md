# Big Tiff StoryForge — Webtool Changelog

> ⚠️ **SUPERSEDED — HISTORICAL REFERENCE ONLY**
> As of 2026-06-22, the live changelog is `CHANGELOG.md` in the GitHub repo
> (`footeprint-prog/big-tiff-storyforge`). This document predates the file
> corruption/recovery event described in `workflow-memory.md` and is kept
> only because it contains useful original spec detail (see Section C of the
> companion `OLD_Implementation_Checklist.md` for the original DraftPad and
> dual-changelog design). **Do not treat entries below as reflecting the
> current state of `writing.html`.** Cross-check anything you need against
> the live file before relying on it.

---


> **Project Reference for Future Conversations**  
> This changelog tracks finalized milestones and major decisions for the Big Tiff StoryForge writing tool.  
> **Always read and update this file** when milestones are completed or significant decisions are made.  
> Last updated: 2026-06-16

---

# Big Tiff StoryForge — Webtool Changelog

**Project:** Big Tiff StoryForge Writing Tool  
**Purpose:** Track finalized progress milestones, major decisions, and completed work on the writing tool. This file is maintained as development progresses.

---

## 2026-06-16

### Milestone: Core Architecture & Planning Complete

**Summary**  
Finalized the foundational architecture for the dynamic Outline system, Safe Sync flow, and supporting features after extensive discussion on user protection, canon alignment, and ADHD-friendly design.

**Key Decisions & Deliverables**
- Established **Safe Sync Philosophy**: Canon is sacred; Erica’s drafts are never overwritten by automation. `review` status requires intentional user action to change.
- Finalized **status values**: Only `unfinished`, `review`, and `complete`.
- Locked the **JSON Schema** for `Big_Tiff_Scene_Structure.json` (the single source Sammy will maintain).
- Created comprehensive **Implementation Checklist** covering Outline, DraftPad, dual ChangeLog system, Library sync, and future Sammy instructions.
- Added specific item for designing gentle prompts/reminders when leaving `review` status scenes.
- Exported both the Checklist and this Changelog as persistent files for ongoing reference.

**Files Created**
- `Big_Tiff_StoryForge_Implementation_Checklist.md`
- `Big_Tiff_StoryForge_Webtool_Changelog.md`

**Next Phase**  
Ready to begin implementation (starting with dynamic Outline renderer and basic sync logic).

---

## Notes for Future Entries

- All entries should be dated and focus on **finalized** milestones or major decisions.
- Include links to relevant files or schema versions when applicable.
- Major UI/UX decisions (especially around `review` status prompts) will be logged here once discussed and implemented.