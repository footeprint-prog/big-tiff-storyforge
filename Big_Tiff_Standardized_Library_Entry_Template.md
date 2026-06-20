# Big Tiff — Library Entry Template

**Version:** 1.0  
**Date:** June 19, 2026  
**Purpose:** This template ensures every Library entry is canon-accurate, useful, and consistently structured. It helps decide what belongs in the Library versus staying only in the Master Story Bible, and keeps the webtool’s Library panel clean, searchable, and inspiring.

---

## 1. Decision Checklist (Before Creating an Entry)

Ask these questions. If most answers are **Yes**, the content probably belongs in the Library.

- Is this a stable canon fact from the current Master Story Bible (not a draft idea or one-off moment)?
- Does it have high re-use value? (Will creators or users need to reference this often?)
- Can it be understood without heavy context or major spoilers?
- Does it fit cleanly into one of the defined categories below?
- Will this entry help someone quickly understand the world or a character without reading the full Bible section?
- Is it self-contained enough to stand on its own in the Library panel?

**If most answers are No** → Leave it in the Master Story Bible (or Character Dossier / Scene Tracker) instead.

---

## 2. Library Categories (Current Structure)

Entries in the Library are organized under categories. The current categories are:

| Category              | What Belongs Here                              | Examples                              | What Does NOT Belong Here          |
|-----------------------|------------------------------------------------|---------------------------------------|------------------------------------|
| **Characters**        | Core + significant supporting characters       | Timmy, The Biscuit Guardian           | One-scene villagers                |
| **Locations**         | Important places with personality or rules     | The Whispering Grove, Morning Kitchen | Generic “a forest”                 |
| **Harmonics & Magic** | The cosmic rules, scales, polarities, how magic works | Resonance Scale, Polarity Laws     | Single spell effects               |
| **Key Objects & Artifacts** | Items with history, power, or recurring importance | The Morning Biscuit, The Harmony Stone | Everyday props                  |
| **Concepts & Cosmology** | Bigger ideas, forces, or world truths       | The Great Balance, Moon Phases        | Minor metaphors                    |
| **Timeline / Key Events** | Major historical or story milestones       | The First Resonance, The Great Split  | Minor daily events                 |
| **Glossary & Terms**  | Special words, names, or phrases that carry weight | Harmonic Resonance, Whispering     | Common words                       |
| **Erica’s Entries**   | Personal entries created by Erica              | (Initially empty)                     | —                                  |

---

## 3. Recommended Entry Structure

Use this structure whether writing in Markdown or populating `Big_Tiff_Library_Entries.json`.

### Markdown Format

```
### [Entry Name]

**Category:** [One of the categories above]  
**Last Updated:** YYYY-MM-DD

**Summary**  
(1–3 clear, canon-accurate sentences. No major spoilers unless the entry is marked as such.)

**Canon Source**  
- Master Story Bible section: [e.g., “Section 3 – The Harmonic Laws”]
- Key quote or detail: [short direct reference]

**Key Relationships**  
- Connected to: [list other Library entries or Bible elements]

**Tags**  
`#tag1` `#tag2` `#cosmic` `#polarity`

**Notes for Creators** (optional)  
Any guidance on tone, usage, or things to keep consistent when writing scenes involving this entry.

---
```

### Alignment with JSON Schema

When adding entries to `Big_Tiff_Library_Entries.json`, map the fields as follows:

- `title` → Entry Name
- `category` → Category
- `lastUpdated` → Last Updated
- `summary` → Summary (what appears in the Library panel preview/list view)
- `content` → Full detailed content (what appears in the expanded view when the user opens the entry)
- `tags` → Tags
- `relatedEntries` / `relatedScenes` → Key Relationships
- `canonical` → Whether this is considered current canon

---

## 4. Creation Process (Step-by-Step)

1. Open the current **Master Story Bible** and go to **File → Version history** to confirm you’re on the latest named version.

2. Identify a strong candidate using the **Decision Checklist** above.

3. Draft the entry using the structure in Section 3.

4. Cross-check against existing Library entries to avoid duplication or overlap.

5. If the entry reveals something new or important, consider whether the Master Story Bible itself needs a small update + new version name.

6. Add the entry to `Big_Tiff_Library_Entries.json` inside the correct category’s `entries` array (entries live under `categories[].entries`, not as flat items with a category field).

7. Name a new version in the Master Story Bible (e.g., `2026-06-19 - Added Library entries for Harmonic Scale + Whispering Grove`).

**Note:** When Aaron or Erica says **“Update webtool”**, you may be asked to create or update Library entries as part of that process. Follow the Guardrails for updating GitHub files when doing so.

---

## 5. Tone & Voice Guidelines

- Keep entries **warm, clear, and slightly wondrous** — matching the spirit of *Big Tiff*.
- Write in **present tense** where possible.
- Avoid heavy plot spoilers unless the entry is intentionally placed in a “spoiler-safe” section later.
- Prioritize **clarity and usefulness** over poetic language (the poetry lives in the Bible and the stories).

---

## 6. Library Changelog & Change Tracking

Individual Library entries do **not** carry review status or `needsReviewReason` flags. Instead:

- A **“Last Updated”** date should be visible in the Library side panel.
- All changes to Library entries should be logged in a **Library Changelog tool** (to be implemented in the webtool). This allows Aaron and Erica to optionally review what changed and why.

When making changes, always record the reason clearly so it can be captured in the Library Changelog.

---

## Related Documents

- `Big_Tiff_Library_Entries.json` — The structured data file that powers the Library panel (entries organized by category).
- `Big_Tiff_Project_Governance_and_Protocols.md` — Highest-level system reference, including the “Update webtool” command and Guardrails for GitHub updates.
- `Big_Tiff_Master_Story_Bible_Version_Control.md` — Protocol for maintaining version history and consistency of the Master Story Bible.

*This template should be used in coordination with the documents above.*