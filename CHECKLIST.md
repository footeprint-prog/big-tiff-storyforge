# Big Tiff StoryForge – Development Checklist

This checklist must be followed for every change made to the project.

## Pre-Change Checklist
- [ ] Read the latest `workflow-memory.md` (especially recent incident logs)
- [ ] Understand the current stable state of `writing.html`
- [ ] Plan the change in small, testable increments

## During Change Checklist
- [ ] Make changes in small, isolated steps
- [ ] Never perform large single-block replacements of HTML + JS together
- [ ] Use safe editing methods (precise string replacement or small targeted edits)

## Post-Change Verification Checklist
- [ ] Run: `python3 validate_html.py writing.html`
  - [ ] Confirm: Div balance passes
  - [ ] Confirm: Expected IDs are present (or note why they aren't)
- [ ] Run: `python3 verify_github_push.py` (when preparing to push)
- [ ] Manually test key functionality (ideally in an actual browser, not just by reading the code):
  - [ ] Panel toggles (Outline & Library buttons)
  - [ ] Side buttons remain visible after clicking
  - [ ] Scene selection from Outline populates guidance correctly
  - [ ] Editor formatting buttons work
  - [ ] Save Draft works
- [ ] Check browser console for errors (especially `SyntaxError` or `ReferenceError`)
- [ ] **"Done" means tested, not just written.** Do not report a feature as complete based on reading the code alone — confirm it actually runs as expected.

## Push Checklist (Mandatory)
- [ ] Read actual file content before pushing (never use placeholder strings)
- [ ] Commit with clear, descriptive messages
- [ ] After pushing, verify raw GitHub URLs contain real code:
  - https://raw.githubusercontent.com/footeprint-prog/big-tiff-storyforge/main/writing.html
  - https://raw.githubusercontent.com/footeprint-prog/big-tiff-storyforge/main/workflow-memory.md
- [ ] Confirm file sizes are reasonable (writing.html > 50KB)
- [ ] Confirm no placeholder text exists in pushed files

## Never Do
- Use placeholder strings when pushing to GitHub
- Make large untested replacements
- Skip verification steps
- Ignore console errors
- Report something as "done" or "verified" without actually checking it in this session

---

## Open Items Carried Forward From Original Spec (2026-06-16)

These were defined in the original, more detailed Implementation Checklist
(now archived as `OLD_Implementation_Checklist.md`) and confirmed via direct
code check on 2026-06-22 to still be **unbuilt**. Carried forward here so
they aren't lost now that the old document is marked historical.

- [ ] **Global Outline/Lore Changelog** — draggable window, same style as
      Notepad/DraftPad, showing Sammy's canon + structure changes. *(Next
      planned feature as of 2026-06-22.)*
- [ ] **Library change detection + alert badge** — Library panel should show
      a visible indicator when canon-driven changes affect Library-sourced
      entries.
- [ ] **Library changelog** — separate from the Outline/lore changelog above;
      tracks changes specifically to `Big_Tiff_Library_Entries.json` content.
      (See `Big_Tiff_Standardized_Library_Entry_Template.md` Section 6.)
- [ ] **Last sync timestamp in Library panel** — currently only shown near
      the Outline; not duplicated in the Library panel.
- [ ] **Auto-expand to first relevant scene** (`unfinished` or `review`) on
      initial load, rather than requiring the user to pick a scene manually
      every session.
- [ ] **Scene reordering must stay disabled** — confirmed NOT present in the
      live file as of 2026-06-22 (hard requirement, verified, no action
      needed unless this changes).

## Parked Ideas / Not Now

A holding pen for things mentioned in passing that aren't urgent. Review
periodically; move items up into active work when ready, or strike them out
if no longer wanted.

- [ ] **Strengthen Sammy's specialized storytelling expertise.** Aaron wants
      to spend dedicated time with Sammy identifying additional foundational
      resources (craft references, structure frameworks, genre-specific
      material) to deepen his story-writing expertise beyond the current
      instruction set. Goal: be able to trust Sammy's editorial judgment more
      fully on scene creation and overall Big Tiff storyline decisions — this
      matters directly to the tool too, since the review-status workflow
      already treats Sammy's `needsReviewReason` calls as authoritative.
      Strengthening his foundation makes that trust better-placed. (2026-06-22)
- [ ] Main writing area layout/visual polish — intentionally deferred until
      after the fantasy-themed visual mockup is complete and broken down into
      skinnable elements (per Aaron, 2026-06-21).
- [ ] Open question: are the `safeGetItem`/`safeSetItem` localStorage wrapper
      functions (mentioned in the 2026-06-21 CHANGELOG entry) still needed?
      They do not currently exist in `writing.html`. Likely a holdover from
      a Grok-sandbox-specific issue that may not apply to Erica's actual
      browser environment — needs a decision, not urgent.

## Questions Needing Erica's Input (Not Yet Reviewed)

See `Erica-Questions-Writing-Tool.md` for the full list compiled during dev
work. Many may already be answered by choices made since 2026-06-21
(per-scene drafts, status selector behavior, leaving-review confirmation) —
this needs a pass to mark which questions are now resolved vs. still open,
rather than assuming the whole list is still live.

---

*Last Updated: 2026-06-22*
