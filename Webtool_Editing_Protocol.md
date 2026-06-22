# Big Tiff StoryForge — Webtool Editing Protocol

**Created:** June 17, 2026  
**Last Updated:** June 17, 2026  
**Purpose:** This protocol ensures that edits to large or complex files (especially `writing.html`) are made reliably, accurately, and with proper verification. It exists to prevent hallucinated progress while keeping development practical and aligned with the project’s calm, supportive values.

This protocol applies to `writing.html` and any future large single-file applications or major refactors.

---

## Core Principles

- **Verification over speed**: Never report work as complete until it has been independently verified.
- **Atomic changes**: Make one focused change at a time on large files.
- **Transparency**: If something is unclear or an edit fails verification, stop and communicate it immediately.
- **Protect the user experience**: All changes should support (not add friction to) Erica’s writing process.

---

## Mandatory Editing Process

When working on `writing.html` (or similar large files), follow these steps:

### 1. Pre-Edit Preparation
- Before planning or making any edit, use `read_file` with `offset` and `limit` to retrieve the **exact current text** of the section being modified.
- Do not rely on memory or previous conversation context for precise string content.

### 2. Make Atomic Changes
- Perform **one small, focused change at a time**.
- Avoid combining multiple unrelated changes into a single `edit_file` call on large files.
- Use the smallest possible `old_string` that uniquely identifies the target.

### 3. Verify Immediately After Every Edit
After **every** `edit_file` or `bash` edit:
- Immediately run verification using `grep` and/or targeted `read_file` calls.
- Confirm the change is present and correct.
- If verification fails, stop and resolve the issue before making further edits.

### 4. Preferred Editing Tools (in recommended order)
1. `edit_file` — Preferred when the target string is short, unique, and was recently read.
2. `bash` + `sed` or `awk` — Useful for inserting larger blocks or when `edit_file` is unreliable.
3. Create a new versioned file (e.g. `writing-v2.html`) only when repeated attempts with the above methods fail.

### 5. Update Project Documentation
After completing a meaningful unit of work:
- Update the relevant item(s) in `Big_Tiff_StoryForge_Implementation_Checklist.md`.
- Add an entry to `Big_Tiff_StoryForge_Webtool_Changelog.md` describing what was done and why.

### 6. Final Delivery
At the end of any editing session on `writing.html`:
- Deliver the updated file to the user using the `render_file` component.
- Provide a short verification summary of what was checked and confirmed.

### 7. Error Handling
- If an edit fails verification more than once, pause and clearly describe the problem to the user before continuing.
- Never assume an edit succeeded without explicit verification.

---

## Scope

**This protocol applies to:**
- `writing.html`
- Any future large, single-file, JavaScript-heavy application files
- Complex refactors or insertions of major new features (DraftPad, ChangeLog windows, Library sync, etc.)

**Normal `edit_file` usage is acceptable for:**
- Small supporting files (Checklist, Changelog, Overview documents, etc.)
- Minor, low-risk changes where verification is straightforward

---

## Integration with Existing Systems

This Editing Protocol works alongside (not instead of) the project’s other systems:

- Always cross-reference the **Implementation Checklist** when planning work.
- Record completed work in the **Webtool Changelog**.
- Follow the broader **Project Overview & Refresh Protocol** when performing full reviews.

---

## Commitment

By following this protocol, we maintain high standards of reliability while keeping development practical and aligned with the project’s goal of providing calm, trustworthy support for Erica.

*This protocol should be referenced at the start of any conversation involving significant edits to `writing.html`.*