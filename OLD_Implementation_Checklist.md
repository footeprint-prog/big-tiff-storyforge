# Big Tiff StoryForge — Implementation Checklist

> ⚠️ **SUPERSEDED — HISTORICAL REFERENCE ONLY**
> As of 2026-06-22, the live, actively-maintained checklist is `CHECKLIST.md`
> in the GitHub repo (`footeprint-prog/big-tiff-storyforge`). This document
> predates the file corruption/recovery event (`workflow-memory.md`) but is
> kept because it contains real spec detail not recorded elsewhere — notably
> Section C (DraftPad + scene-level Change/Review window) and Section D
> (Library change detection + Library-specific changelog), both partially or
> fully unbuilt as of this notice. Unbuilt items have been carried forward
> into the live `CHECKLIST.md` "Parked Ideas / Not Now" and "Open from
> original spec" sections so they aren't lost. Treat checkboxes below as
> historical intent, not current status — verify against the live
> `writing.html` before assuming anything here is built.

---


> **Project Reference for Future Conversations**  
> This is the master task list for the Big Tiff StoryForge writing tool.  
> **Always read this file** when the user mentions the webtool, Outline, DraftPad, sync features, or Sammy integration.  
> Use it to track progress and check off completed items.  
> Last updated: 2026-06-16

---

# Big Tiff StoryForge — Implementation Checklist

**Project:** Big Tiff StoryForge Writing Tool  
**Date:** June 16, 2026  
**Purpose:** Master checklist for building the ADHD-friendly writing tool. This document captures all planned features, protections, and future work related to the dynamic Outline, Safe Sync system, DraftPad, changelogs, and Sammy integration.

---

## A. Data & Sync Layer

- [ ] Finalize and host `Big_Tiff_Scene_Structure.json` in Google Drive (public link sharing on this file only)
- [ ] Implement `fetchLatestFromCanon()` + parsing + change detection in `writing.html`
- [ ] Automatically set `status: "review"` + populate `needsReviewReason` when relevant changes are detected
- [ ] Generate noticeable **Sync Summary Report** on every successful sync
- [ ] Maintain global `syncChangeLog` (accessible from Outline panel)
- [ ] Add last sync timestamp in both Outline and Library panels

## B. Outline Panel

- [ ] Replace static HTML with dynamic renderer driven by the JSON schema
- [ ] Auto-expand to the first relevant scene (`unfinished` or `review`) on load
- [ ] Use succinct status badges only: **Unfinished**, **Review**, **Complete**
- [ ] Add calm “Sync with Canon” button
- [ ] Provide access to global Outline Changelog (draggable window)
- [ ] Clear visual treatment for scenes in `review` status

## C. Writing Page / Scene Experience

- [ ] Add **“Store Draft”** button that saves current editor content to the scene’s local `storedDrafts` array
- [ ] Build **DraftPad** — scene-local draggable window (similar aesthetic to Notepad) for managing multiple stored drafts per scene
- [ ] Add **“Change / Review”** button near the prompt that opens the scene-specific **ChangeLog** window
- [ ] Scene-specific changelog entries are created/updated when that scene is affected by a sync
- [ ] **Design gentle prompt or reminder when leaving a scene that is currently in `review` status**  
  **Context:** Simply viewing the ChangeLog does not count as having reviewed the changes. Erica must intentionally decide the new status. She may leave it as `review` (personal reminder), change to `unfinished` (plans to rewrite), or `complete` (satisfied with current draft). The prompt should feel supportive, not pressuring. Discuss UI pattern and wording during UI development.

## D. Library Panel

- [ ] Detect canon-driven changes to Library-sourced entries
- [ ] Display alert/badge when Library content has been updated
- [ ] Add last sync timestamp in Library panel
- [ ] Provide accessible Library changelog

## E. Protection & Behavioral Rules (Hard Requirements)

- [ ] Draft text and stored drafts are **never** modified by sync logic
- [ ] Scene movement/reordering is not user-controllable in the tool
- [ ] `review` status is set automatically but requires intentional user action to change

## F. Polish & Experience

- [ ] All new windows (DraftPad, ChangeLog) match existing calm, draggable style
- [ ] Summary reports and flags are noticeable without being overwhelming
- [ ] Focus mode remains fully functional
- [ ] Badges and UI elements remain compact enough for the left panel

## G. Agent & System Maintenance (Return to Later)

- [ ] Create comprehensive instructions for StoryForge Sammy (and future copies) covering:
  - Responsibility to maintain `Big_Tiff_Scene_Structure.json`
  - Format rules and update discipline
  - How to write high-quality scene prompts (include who is present, location, what the draft must contain, and contextual links to previous/next scenes)
  - Protocol for proposing scene movements or major structural changes (must be discussed with Aaron & Erica first)
  - Generating clear sync summary reports
  - Coordination with DraftPad and dual changelog systems
  - Guidance on Library category structure when relevant

---

**Notes:**
- This checklist should be referenced before and during implementation phases.
- Items in Section G are intentionally deferred until core tool functionality is stable.
- The `review` status behavior and gentle exit prompts (Section C) require discussion during UI work.