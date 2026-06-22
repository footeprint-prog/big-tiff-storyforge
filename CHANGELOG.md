# Big Tiff StoryForge – Changelog

All notable changes to the webtool during active development.

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
