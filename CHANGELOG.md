# Big Tiff StoryForge – Changelog

All notable changes to the webtool during active development.

## [2026-06-26] – Writing Area Layout Redesign, Floating Guidance Cards, UPDATES Merge (Claude session)

All items below were verified with a real headless-browser test in-session
(Playwright), not just read/written. See test transcripts in session history
if a re-check is ever needed.

### Added
- **Fixed-width writing area layout** — center content now has a permanent
  max-width matching the clearance the side panel buttons already have to
  the screen edge (button's own 24px+78px clearance, mirrored), so there's
  always deliberate resting space on either side regardless of panel state.
  Replaces the old full-bleed layout.
- **Scene Status Control Bar** — new full-width bar, sticky-capped directly
  beneath the (also now sticky) Scene Summary bar. Houses the status pill,
  autosave text, Save Draft, Draft Log, and a Send-to-Sammy button. Styled
  with a 5px gold top border (matching the side panel buttons) and a 1px
  full-opacity gold bottom border, filled in the Library panel's green.
- **Send to Sammy (real implementation)** — builds a formatted export
  (editorial instructions + scene title/summary/draft) into a draggable,
  copy-to-clipboard window. Convenience export only; never calls any API or
  sends anything automatically. Explicit Draft Protection language included
  so it can't be misread as "rewrite this for me."
- **Proofreader** — mechanical spelling/grammar/tense pass via the free
  LanguageTool API, in its own floating window. Apply / Apply All only ever
  touch text the user has seen and approved.
- **Custom status-pill dropdown** — replaces the native `<select>` (which
  always duplicated the current value inside its own open option list).
  Button + slide-down menu, color-matched to the outline badges.
- **Floating guidance cards** — each of the 6 guidance boxes (Purpose, Key
  Canon, Emotional State, Beats, Setting & Tone, Success) can now float into
  its own small, lightweight draggable/resizable window via a corner icon.
  Single source of truth — the real DOM node moves, nothing is ever copied
  or kept in sync. Floating state is explicitly temporary: auto-resets on
  canon re-sync, on init/reload, on Focus mode, and now also when Notepad or
  Draft Log is opened.
- **Minimize/maximize toggle** on home-base guidance cards — collapses a
  card in place to a header-only row (independent of floating).
- **Editor display zoom** (−/100%/+) — CSS-only font-size control on the
  editor; never touches saved draft content.
- **Outline title shortening** — long scene titles truncate at the en-dash
  in the compact outline view only; full titles still shown elsewhere
  (Draft Pad, scene label, Changelog).
- **Outline expand/collapse persistence** — re-renders (e.g. from a status
  change or re-sync) no longer collapse Act/Chapter sections the user had
  open.
- **Library review-flag markers** — categories and individual entries
  currently flagged for review by Sammy show a small "!" marker (with
  tooltip reason at the entry level), so changes are visible without
  expanding every category.
- **Changelog icon buttons** added directly to the Outline and Library
  panels, each pre-filtering the shared Changelog window to the relevant
  type ('scene' / 'library') when opened from that panel.

### Changed
- Editor base font reduced one step; toolbar reorganized (Proofreader now
  lives with text tools + word count; Save Draft/Draft Log/Sammy moved into
  the new Scene Status Control Bar).
- Center writing area now has its own independent scrollbar, decoupled from
  the side panels.
- Removed the standalone top-header Changelog button (superseded by the
  panel-specific, pre-filtered icon buttons above).

### Fixed
- `.notepad.hidden { display: none !important }` CSS guard was previously
  dead code — `openNotepad()` / `openDraftPad()` / `openChangelog()` never
  actually removed the `.hidden` class, so the guard had no effect. Now
  fixed in all three.
- A genuine 0.5px hairline gap between the Scene Summary and Status Control
  Bar, caused by `offsetHeight` rounding a fractional height — fixed by
  switching the sticky-offset calculation to `getBoundingClientRect()`.
- Side panel buttons briefly regressed off their correct position during
  the layout work (root cause: the outer flex container needed explicit
  `w-full` to hold its `max-w-screen-2xl` cap once its children gained more
  specific sizing) — caught and fixed same-session via direct measurement.

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
