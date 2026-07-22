# Big Tiff StoryForge – Changelog

All notable changes to the webtool during active development.

## [2026-06-22] – Send to Sammy: Real-World End-to-End Confirmation

### Validated
- **First real-world test of "Send to Sammy" against an actual Sammy
  conversation**, not just simulated UI checks. Aaron pasted the exported
  block (instructions + scene context + draft text) into a real Sammy
  session and shared the result. Confirmed working correctly end-to-end:
  - Sammy respected the Draft Protection instruction exactly as written -
    flagged issues and explained reasoning rather than rewriting Erica's
    prose.
  - Sammy asked a clarifying question ("Would you like me to note this as
    a prompt-alignment issue for the tool?") rather than silently assuming
    how to handle an ambiguous case - a good sign the instructions are
    being interpreted as intended.
  - Feedback included exactly the kind of contextual catch a grammar tool
    can't produce (scene summary vs. draft mismatch, a missing character
    beat) - validating the original reasoning for building this as a
    Sammy-routed feature rather than folding it into the Proofreader.
  - The scene/title mismatch Sammy flagged was an artifact of Aaron's own
    test placeholder text (not matched to real scene guidance) - expected,
    not a tool bug.

### Also Fixed (same session)
- **Critical bug: closed floating panels could intercept clicks on whatever
  was visually behind them**, causing what looked like an "endless wait"
  on a second Proofreader check after using Apply All. Root cause: `.notepad`'s
  own `display: flex` rule and Tailwind's `.hidden` utility have equal CSS
  specificity, so whichever stylesheet happened to parse last in the
  cascade could win - meaning a "hidden" panel (e.g. the unopened Send to
  Sammy window) could still be rendered and clickable-through depending on
  load timing. Fixed with an explicit `!important` override on
  `.notepad.hidden`, plus corrected all five panel-open functions (Notepad,
  Draft Pad, Changelog, Proofreader, Send to Sammy) to properly remove the
  `hidden` class on open instead of only masking it with an inline style.
  Verified by reproducing the exact reported sequence (Apply All → Check
  Draft again) - previously timed out after 30s, now completes instantly
  with fresh results.

---

## [2026-06-22] – Status Selector Rebuilt as Custom Widget

### Changed
- **Replaced the native `<select>` status selector with a fully custom
  widget** (button trigger + absolutely-positioned sliding menu). Root
  cause for the rebuild: native `<select>` elements always show the
  closed-state value a second time as the first row of their own open
  option list - this is standard browser behavior with no CSS workaround,
  so a real fix required dropping the native element entirely. Confirmed
  the swap was low-risk first by checking that `#scene-status-select` was
  only ever referenced by `updateStatusSelector()` itself - nothing else in
  the codebase (including `renderDynamicOutline`) touches it directly, so
  preserving the same function name and call contract meant no other code
  needed to change.
- New widget: clicking the current-status pill flattens its bottom corners
  and a connected menu slides open directly beneath it, showing ONLY the
  valid alternative statuses (never a duplicate of the current one, never
  "Review" as a pickable option unless it's already current - same
  review-lock safety rule as before, fully preserved). Each menu option is
  colored to match its corresponding outline badge. Click-outside-to-close
  added.

### Fixed
- **"Unfinished" text was clipped by the arrow icon** in the previous
  native-select version - caught via an actual screenshot, not just
  computed-style checks (which had falsely looked fine). Resolved as part
  of the full widget rebuild (no separate native-select patch was viable).
- **Dropdown arrow was invisible specifically on the Review status** - the
  default gold arrow icon was the same color as Review's own gold
  background. Caught by screenshotting all three states side by side.
  Fixed with a dark-red arrow variant used only for the Review state.

### Process Note
- This round relied heavily on actual Playwright screenshots, not just
  computed-style assertions - several real bugs (text clipping, the
  invisible arrow) were only visible that way and would have shipped
  unnoticed otherwise. Worth continuing as standard practice for any
  future visual/layout work in this sandbox.

---

## [2026-06-22] – Header Cleanup: Changelog Access Moved to Panels

### Changed
- **Removed DRAFTS and CHANGELOG text buttons from the top app header.**
  Draft Pad access now lives only in the editor toolbar ("DRAFT LOG"
  button, added earlier today). Changelog access moved to two new icon
  buttons (see below) rather than a standalone header button.
- **Sync Canon button (outline panel) reduced from full-width to `flex-1`**,
  sharing its row with the new changelog icon button instead of spanning
  the whole panel width alone.

### Added
- **Changelog icon button (clock-rotate-left) in the outline panel**,
  next to the Sync Canon button.
- **Same changelog icon button added to the Library panel header**, next
  to the entry count badge.
- Both verified to actually open the Changelog window via real click
  tests; confirmed exactly two such buttons exist (no duplicates), and
  Sync Canon / Draft Log functionality both still work correctly after
  the layout change.

---

## [2026-06-22] – Outline Title Shortening, Badge/Layout Fixes

### Added
- **Shortened scene titles in the outline panel.** Sammy's scene titles
  often include a descriptive suffix after an en-dash (e.g. "Morning
  Chores & The Wobbling Plate – First Odd Occurrence"). New
  `shortenSceneTitleForOutline()` helper strips everything from the en-dash
  onward for outline display only, reducing wrapping/bloat. Other places
  that show the full title (Draft Pad header, writing-area scene label,
  Changelog entries) intentionally keep the full descriptive title.
  Verified against the real example plus edge cases (no dash, multiple
  dashes, empty string) and confirmed the full title is still preserved
  everywhere outside the outline.

### Fixed
- **Status badges still mismatched in size after the first fix.** The
  font-weight fix earlier today was real but incomplete - the actual
  reported symptom (badge height tied to how many lines the scene title
  wrapped to) was a layout issue, not a font-weight issue. Root cause: the
  scene row was `flex justify-between` with no `align-items` set, so a
  multi-line title would stretch its sibling (the badge) to match height
  by flex's default `stretch` behavior. Fixed by setting `items-start` on
  the row and `align-self: flex-start` + explicit `flex-shrink: 0` directly
  on the badge.
- **`.scene-status` (outline) and `.scene-status-dropdown` (editor) could
  drift in size independently**, since they were two separately-maintained
  CSS rules. Unified under one shared block for font-size, min-width,
  line-height, and font-weight, so they can't diverge again; only
  background/text colors differ by status now. Verified via direct
  computed-style comparison (trustworthy here since these are custom CSS
  classes, not Tailwind utilities).

### Process Note
- **This sandbox cannot load Tailwind** (CDN script requires network access,
  unavailable here), which silently invalidated an earlier round of
  "verified" layout tests - they were measuring unstyled markup without
  realizing it. Custom CSS classes (like `.scene-status`) are unaffected
  and remain reliably testable; Tailwind-utility-dependent layout (panel
  widths, positioning, text wrapping) is NOT reliably testable in this
  environment and requires Aaron's confirmation in a real browser. Flagged
  explicitly rather than implied as verified.

---

## [2026-06-22] – Outline Status Badges Now Uniform Size

### Fixed
- **Review status badge was visibly larger than Unfinished/Complete in the
  outline.** Root cause: `.scene-review` carried `font-weight: 600` while
  the other two statuses used `500`, all under the same `.scene-status`
  base class with otherwise identical font-size/padding/min-width. Bold
  text at the same size renders larger. Dropped `.scene-review` to
  `font-weight: 500` to match. Verified by measuring actual rendered
  bounding boxes in a real browser test (not just inspecting CSS): all
  three badges now measure identically at 90×15px with matching font-weight.

---

## [2026-06-22] – Outline Expansion State Persists Across Status Changes

### Fixed
- **Outline panel collapsed itself on every status change.** `renderDynamicOutline()`
  rebuilt the entire Act/Chapter tree from scratch on every call (including
  the one `handleStatusChange` triggers after a confirmed status change),
  and every section defaulted back to collapsed with no memory of what the
  user had open. Reported directly by Aaron after the status-selector work
  shipped. Fixed by capturing which Acts/Chapters are currently expanded
  (via `data-act-key` / `data-chapter-key` attributes) immediately before
  the rebuild, then re-applying that exact expansion state afterward.
  Verified with a real browser test: expand Act 1 → Chapter 1, change a
  scene's status, confirm the outline stays open at the same depth and the
  status badge updates live. Also verified the inverse - sections left
  collapsed stay collapsed and aren't accidentally force-expanded as a
  side effect. (One early test gave a false negative due to an unreliable
  Playwright text-selector click, not an actual bug - caught by checking
  real DOM state directly rather than trusting the first result.)
- Confirmed directly by Aaron in the live tool.

---

## [2026-06-22] – Library Canon Sync: Real-World Validation + Review Marker

### Validated
- **First real-world test of Library canon sync**, against actual data
  Sammy pushed via "Update webtool" (not simulated test data). Aaron
  confirmed directly: Library panel showed correct real entries, Changelog
  "Library" filter showed the logged changes, last-sync timestamp updated
  correctly, and no console errors. This is the first end-to-end
  confirmation of the full Library sync pipeline built earlier today.

### Added
- **Review marker on Library category headers** — a "!" now also appears
  before any category name that contains at least one flagged entry,
  hovering lists which entries inside changed (e.g. "Updated: Tiff,
  Timmy"). Combined with the per-entry marker, this closes the "Library
  change detection + alert badge" checklist item — verified with a real
  browser test covering multiple categories, mixed flagged/unflagged
  entries, and multiple flags in one category.

- **Review marker on Library entry titles** — a small "!" now appears
  before the title of any Library entry currently flagged for review by
  Sammy (`status: "review"` + `needsReviewReason` set). Hovering shows the
  reason as a tooltip. Uses the same signal already driving the Library
  Changelog filter, so the two stay consistent. Verified with a real
  browser test against mixed flagged/unflagged entries.

### Closed
- ~~Library panel still has no panel-level badge/count~~ — resolved via
  the two-level marker system above rather than a single count badge.

---

## [2026-06-22] – Status Selector Fix, Sync Reconciliation, Draft Pad (Claude session)

All items below were verified with a real headless-browser test in-session
(Playwright), not just read/written. See test transcripts in session history
if a re-check is ever needed.

### Added
- **Status selector UI** — dropdown next to scene title, styled to match
  outline badge pills (`scene-complete` / `scene-unfinished` / `scene-review`
  classes reused). "Review" only appears as an option when it's the scene's
  actual current state; the user can never pick it back in deliberately.
- **Review Note box** — displays `scene.needsReviewReason` below the Scene
  Summary, visible only while status is `review`.
- **Confirm prompts**: (1) when changing a scene off `review` status,
  showing Sammy's reason text; (2) when navigating away from a scene still
  in `review` without changing its status. Both allow the user to back out.
- **Per-scene status persistence** (`bigtiff-scene-statuses` in
  localStorage) — user-set statuses now survive re-syncs and page reloads,
  since the canon JSON itself stays read-only/untouched by the tool.
- **Per-scene draft autosave** (`current-draft:<sceneId>`) — replaces the
  old single global `current-draft` key, which caused drafts to bleed
  between scenes.
- **Draft Pad** — new draggable/resizable window (DRAFTS button, top bar),
  scoped per-scene. Shows the live "Current Draft" plus user-created named
  snapshots (full-rewrite checkpoints). Snapshots can be renamed, restored
  into the editor (with confirm), and deleted (with confirm). Verified
  scene-scoping is fully isolated — no cross-scene bleed.

### Changed
- `reconcileSceneStatuses` (replaces `detectAndApplyReviewFlags`) — review
  status is now Sammy's editorial call, read directly from his `status` +
  `needsReviewReason` fields in the JSON, rather than the tool diffing text
  itself. First-time-seen scenes with no Sammy flag default to `review`
  (not `unfinished`) since an unseen scene is, by definition, unreviewed.

### Fixed
- **Duplicate `updateStatusSelector` function** — a second, broken stub
  definition later in the file was silently overriding the working one,
  so the dropdown never actually locked Review status or refreshed the
  outline. Removed.
- **Dead `currentCanonData` guard in `handleStatusChange`** — caused every
  status change to silently no-op whenever `currentCanonData` was unset.
  Found via browser test, not code reading. Removed.
- **Global (non-scene-aware) draft autosave** — previously one shared
  localStorage key for all scenes; writing in Scene 2 could overwrite or
  be overwritten by Scene 1's draft. Now scene-scoped.

### Verified Still Absent (checked directly, not assumed)
- `safeGetItem` / `safeSetItem` localStorage wrappers — mentioned in the
  2026-06-21 entry below, but do not exist anywhere in the file. Open
  question in `CHECKLIST.md` Parked Ideas.
- Scene drag/reorder capability — confirmed absent, which is correct per
  the hard requirement in the original spec (Section E).

### Documentation
- Reconciled duplicate checklist/changelog pairs. The older, more detailed
  `Big_Tiff_StoryForge_Implementation_Checklist.md` and
  `...Webtool_Changelog.md` (2026-06-16) are now marked superseded/historical
  at the top of each file, with still-relevant unbuilt items (global
  Outline/Lore Changelog, Library changelog + alert badge, auto-expand to
  first scene) carried forward into the live `CHECKLIST.md`.

---

## [2026-06-21] – Initial Launch Appearance Complete

### Added
- Full flanking guidance layout in center writing area (Purpose, Key Canon, Emotional State, Must-Happen Beats, Setting & Tone, Success)
- Scene label and status selector when a scene is loaded
- `populateGuidance(scene)` function
- `updateStatusSelector(scene)` function
- Enhanced `selectSceneFromCanon(scene)` function
- Safe localStorage wrappers (`safeGetItem` / `safeSetItem`) to prevent crashes in sandboxed environments
- GitHub Actions workflow (`.github/workflows/verify-push.yml`) for push content verification
- `HANDOFF.md`, `CHECKLIST.md`, and `workflow-memory.json` for project handoff

### Changed
- Panel system converted to `fixed` positioning so panels sit below the header and align properly with window edges
- Side panel buttons now remain visible and functional at all times (no longer hidden when panels are open)
- Sync success message now appears inside the left panel instead of as a global toast

### Fixed
- Multiple HTML structure imbalances caused by large replacements
- JavaScript `SyntaxError` from corrupted regex literals in `populateGuidance()`
- `SecurityError` when accessing `localStorage` in sandboxed environments
- Side buttons disappearing after being clicked

### Process Improvements
- Moved from unreliable placeholder-based pushes to proper local git workflow
- Added mandatory verification steps after every change
- Created comprehensive process documentation in `workflow-memory.md`

---

## Earlier Development (Pre-2026-06-21)

Significant earlier work included:
- Initial single-file HTML architecture with Tailwind + Font Awesome
- Dynamic Outline rendering from canon JSON
- Draggable/resizable Notepad system
- Library system with localStorage persistence
- Focus Mode
- Basic scene status management

*Detailed history is preserved in `workflow-memory.md` under the Incident Log sections.*

---

*This changelog focuses on major milestones during active collaborative development.*
