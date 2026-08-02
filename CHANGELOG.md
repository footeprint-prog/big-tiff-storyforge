# Big Tiff StoryForge – Changelog

All notable changes to the webtool during active development.

## [2026-08-03] – Autosave text embedded in the editor, not a floating overlay

Aaron reported two problems with the mobile autosave/word-count text: it
needed to be selectable-proof and visually "embedded... on the same layer"
as the writing field rather than floating over the page, and it was
rendering on top of the Library/Outline side panels once they slid open.

### Changed
- **`#auto-save-status` is now desktop-only.** The several rounds of mobile-
  specific repositioning on this element (2026-07-24 through 2026-08-02) are
  gone - it's simply `display:none` on mobile now. Desktop is completely
  unaffected: same element, same inline position in the status bar, same
  behavior.
- **New `#mobile-draft-ghost` element** takes over the job on mobile,
  positioned `absolute` inside `#editor-column` (given `position:relative`
  for this purpose) so it renders at `#editor`'s own bottom-right corner
  instead of a fixed viewport coordinate. Because the editor is naturally
  hidden/covered by the Library and Outline panels when they're open (same
  as any other content behind them), the ghost is now covered along with
  it automatically - this isn't a z-index fix, it's a structural one: the
  ghost only exists in the same screen region as the editor, so whatever
  covers the editor covers it too.
- **Deliberately a SIBLING of `#editor`, never a descendant.** `#editor` is
  `contenteditable="true"`; a descendant node inside a contenteditable
  region is selectable/copyable/editable by native browser behavior more
  or less regardless of `user-select`, and worse, risks ending up captured
  by `editor.innerHTML` - the actual saved draft, sent to Sammy, pushed to
  the repo. Living as a sibling, visually overlaid via absolute positioning,
  gets the "embedded in the field" look with zero risk of ever touching
  real draft content. Also carries `pointer-events: none` and
  `user-select: none` as defense in depth, though the structural separation
  is the real guarantee.
- **`getAutosaveStatusEl()`** (new) centralizes the mode-dependent target -
  `#mobile-draft-ghost` on mobile, `#auto-save-status` on desktop - so
  `saveDraft()`'s and `goToProofreaderIssue()`'s flash messages, and
  `updateWordCount()`'s merged-count write, don't each need their own
  `isMobileMode()` branch. All three now go through it.
- **`--autosave-gap` removed** (was `0.9rem`) - its last real use (this
  element's own `bottom` offset) is gone now that the element is
  `display:none` on mobile; its other use (the rail's boundary clamp) was
  already removed the day before. Removed rather than left defined-but-
  unused, and the two comments that still described it as "in use" are
  corrected.

### Verification notes
Measured at 440×956 with a seeded scene:
- Ghost's rect confirmed fully inside `#editor`'s own rect, and confirmed a
  genuine DOM sibling (`editor.contains(ghost)` false) rather than a
  descendant.
- Selected all content within `#editor` (`Range.selectNodeContents` +
  `Selection`) and confirmed the resulting selected text and
  `editor.innerHTML` never contain the ghost's text - the "not selectable
  or copied" requirement verified directly, not just asserted from the
  structural argument.
- Opened the Library panel and confirmed via `elementFromPoint` at the
  ghost's exact screen location that the panel (not the ghost) is topmost
  there - the reported bug is gone.
- Flash-guard re-verified through the new helper: `saveDraft()`'s flash
  survives a mid-flash keystroke intact and the eventual restore holds the
  correct pre-flash snapshot, same as before, now via
  `getAutosaveStatusEl()` on both branches.
- Re-ran the existing gesture suite (rail tap/scrub, status-arm drag, Focus
  collapsing both compound controls) - all still pass; none of this touched
  their code paths, but the shared helper function and the removed CSS
  property were both worth confirming nothing else broke.
- Desktop re-verified explicitly (`isMobileMode()` reset first): `#auto-
  save-status` still `display:flex`, same text/rect as always, same flash
  behavior via the new helper; `#mobile-draft-ghost` `display:none`;
  `#editor-column`'s `position` still `static` (the `position:relative`
  rule is mobile-scoped only).

## [2026-08-02] – Rail no longer aligns with the handle; Focus resets both compound controls

Four adjustments at Aaron's direction, all mobile-only, same day as the
round below. Supersedes that round's `--autosave-gap` pullback fix for the
same corner-glitch bug, with a different mechanism per his follow-up ask.

### Changed
- **Unrestricted the handle's bottom slide again** - `getRailBand()`'s
  bottom reverts to the true, flush `window.innerHeight - navClear` (no
  longer pulled back by `--autosave-gap`). That property is untouched and
  still drives `#auto-save-status`'s own position; only its use as a rail-
  boundary buffer is gone.
- **The expanded strip no longer clamps into the same band the handle
  uses.** This is the real fix for the corner-glitch Aaron reported a
  second time ("still shows a graphics glitch... due to the rail's rounded
  corner"): the OLD fix (pulling the boundary back) stopped the handle
  short of true-edge, but the STRIP's own clamp still forced it to match
  wherever the handle stopped - two differently-radiused corners (the
  strip's 8/10px, the handle's 12px) landing at the identical Y, which is
  what actually reads as a glitch regardless of where that Y is. Per
  Aaron's fix: "the rail cannot bottom or top align with handle... the rail
  opens under the nav button row... under the header." The strip's position
  is now purely `handleTop + handleH/2 - railH/2`, uncapped - when a near-
  edge handle's centering math pushes it past headerH/navTop, it's now
  allowed to, extending into that territory rather than stopping at it.
- **`#mobile-nav` and `.fantasy-header` z-index raised to 480** (mobile
  only; desktop keeps nav's prior stacking and the header's shared `z-50`),
  above the rail (465) and its handle (470). This is what makes "opens
  under" literal: the strip's now-unclamped overflow renders BEHIND these
  two permanent chrome bars, whose own opaque backgrounds and existing
  borders (nav's top border, header's bottom border) cap the overflow with
  a single clean straight edge - not the strip's own mismatched rounded
  corner. Confirmed nothing else reaches into header/nav's territory today
  (left/right panels, the scrim, open drawers all already self-limit to
  `bottom: var(--mobile-nav-clear)` or an equivalent max-height), so this
  only changes behavior for the rail.
- **The handle's offset within the rail is now a fixed constant**
  (`(railH - handleH) / 2 - borderTop`), not clamped - a direct
  simplification that falls out of the strip no longer being independently
  clamped: since the strip's position always exactly tracks the ideal
  centered value now, the handle's placement within it never varies,
  removing the min/max clamp that existed only to cover the old clamped-
  divergence case.
- **`#auto-save-status`'s `right` tightened 1.6rem → 1rem.** Aaron: "autosave
  text line must appear under the handle+rail." Measured that the old value
  left the text's own right edge 1.6px short of the COLLAPSED handle's left
  edge (24px wide, flush right) - a near-miss, not a real overlap, in that
  one state (the expanded handle, sitting further left, already cleared it
  comfortably). 1rem guarantees ≥8px of real overlap in both states, on any
  viewport width (the two `right`-from-edge offsets cancel viewport width
  out of the comparison). Reverified it still clears the editor's own
  rounded border.
- **Focus now also collapses the guidance rail and closes the scene-status
  arm's action bank.** `mobileCloseEverything(null)` (which Focus already
  called) never touched either - a new `collapseGuidanceRail()` (idempotent,
  mirrors the existing `resetStatusActionsBank()`) plus a call to that
  existing function are both now wired into `mobileNav('focus')`, so Focus
  is genuinely "back to the calmest possible screen" rather than leaving
  either compound control open.

### Verification notes
Measured at 440×956 with a seeded scene, real `PointerEvent` sequences:
- Dragged the handle to both true extremes in the collapsed state - lands
  exactly flush (36px top, 878px bottom on this viewport), unrestricted.
- Expanded from each extreme: the strip's own edge now differs from the
  handle's edge by >100px in both directions (was 0, the glitch condition),
  while the handle itself stays exactly at the true boundary throughout.
- `elementFromPoint` at the exact seam in both directions, and again deeper
  inside each bar's own territory, returns `#mobile-nav`/`.fantasy-header`
  as topmost in every case - confirmed the strip's overflow is genuinely
  hidden behind them, not floating on top.
- Autosave overlap re-checked in both collapse states after the `right`
  fix: real geometric overlap confirmed in both, `elementFromPoint` at the
  overlap returns the handle (`mobile-rail-toggle`) as topmost in both,
  and the text's right edge still sits inside the editor's own right edge
  (no new collision with the border it was originally pulled in to avoid).
- Focus tested with both the rail expanded AND the status-arm's action bank
  open simultaneously - one call correctly collapses/closes both.
- Re-ran tap-to-open, drag-to-switch, and the unrelated status-arm drag -
  all still pass after this round's rewrite.
- Desktop A/B against pristine `HEAD` at 1280×800 (isMobileMode() explicitly
  reset first - this environment's `navigate` can carry the forced-preview
  flag across loads): header z-index still 50, header rect unchanged, nav/
  rail still `display:none`, `#auto-save-status`'s `right` still `auto`
  (unset) - all mobile-only changes confirmed inert on desktop.

## [2026-08-02] – Handle taller, bottom clamp pulled back, autosave/word-count merged

Six adjustments at Aaron's direction, all mobile-only. Desktop A/B-verified
against pristine `HEAD` throughout - see verification notes.

### Changed
- **Handle height 96px → 112px** (`calc(2 * var(--rail-tab-h))`), "as tall
  as 2 rail-drawer boxes" per Aaron - a rail-tab (the box that opens a
  drawer) is 3.5rem/56px, so two of them is 112px. This deliberately breaks
  the 4:1 iOS-PiP-handle ratio matched exactly two rounds ago; Aaron's new
  explicit number supersedes that fidelity goal. Width (24px) and the 12px
  corner radius are unchanged - the radius is derived from width, not
  height, so it's still a true semicircle at the new size.
- **Rail's bottom drag boundary pulled back from true nav-top.** For one
  round (earlier the same day) the band was flush at both ends. Aaron
  flagged a real visual problem at the true bottom: the rail's own rounded
  corner (10px on the wrapper, 12px on the handle) meets the dead-straight
  top edge of the bottom nav bar with nothing to round into - a visible
  notch. Rather than add a dock-state radius override (the fix the
  scene-status arm uses for its own corner-morph docking), the simpler ask
  was to stop the handle short of the true edge. New `--autosave-gap` custom
  property (`0.9rem`) is shared between `#auto-save-status`'s own `bottom`
  offset and `getRailBand()`'s JS clamp, so the two can't drift apart - the
  rail's new bottom limit is deliberately the SAME line the autosave text
  sits on, not an arbitrary buffer.
- **The handle+rail now overlay the autosave/word-count line** at their new
  bottom maximum - a direct consequence of the clamp change above (the
  112px-tall handle spans well past the autosave line's own ~19px height
  once its bottom edge is pinned to that line), not a separate mechanism.
  `#auto-save-status` stays z-index 30, well under the rail's 465, so the
  rail already paints over it wherever they overlap.
- **Word count folded into the autosave line**: mobile now shows "N words
  autosaved" instead of "Auto-saved just now", updated by the same
  `updateWordCount()` that already drove the toolbar counter. Desktop is
  unaffected - it keeps `#word-count` and `#auto-save-status` as two
  independent elements exactly as before; the new write is gated on
  `isMobileMode()`.
- **Original toolbar word counter removed on mobile.** Hides
  `#word-count-wrap` (the wrapping div, added this round), not just the
  `#word-count` span inside it - hiding only the span would leave an
  empty-but-present flex child still consuming its share of the row's
  `gap-x-3` spacing on both sides.
- **Zoom scaler (−/100%/+) capped at 44px total height**, matching its
  sibling editor-toolbar buttons (B/I/U/Proofreader, all `--tap`/44px) -
  it had been running ~4px taller than them due to its own border+padding
  stacking on top of the 44px its child buttons already establish.
  `box-sizing: border-box` on the new `#editor-zoom-control` id makes the
  border draw inside the fixed 44px rather than adding to it; the inner
  buttons' `min-height` is overridden to `auto` so they don't fight the
  parent's fixed height with their own independent 44px floor - this
  renders the −/+ buttons a few px under the project's touch-target floor
  in isolation, a small, deliberate, Aaron-specified exception (same
  precedent as the 24px-wide handle already documented in
  `BUTTON_STYLE_GUIDE.md`), not an oversight.

### Fixed (found during this round, not a separate ask)
- **A real race** in `updateWordCount()`'s new mobile write: `saveDraft()`
  and `goToProofreaderIssue()` both show a temporary flash message on
  `#auto-save-status` ("Draft saved!", "Couldn't find that text") and
  restore a *snapshot* of the prior text after a timeout. Without a guard,
  typing during that window would both cut the flash short (overwritten by
  a word count) and leave the eventual restore holding a stale count (the
  snapshot taken before typing resumed). Fixed with a shared
  `autoSaveFlashActive` flag, set for the duration of both flash sites and
  checked before every mobile word-count write.

### Verification notes
Measured at 440×956 with a seeded scene, `getBoundingClientRect()` /
`getComputedStyle()`, real `PointerEvent` sequences:
- Handle height exactly 112px in both collapse states.
- Zoom control exactly 44px total (was ~48px), `box-sizing: border-box`
  confirmed via computed style.
- Dragged to the extreme bottom in both collapsed and expanded states: the
  handle's/strip's bottom edge lands exactly at the new band boundary (not
  the old true nav-top), genuinely overlaps `#auto-save-status`'s rect, and
  provably does NOT reach the true nav-top edge. Top extreme re-confirmed
  still flush (unaffected - only the bottom changed this round).
- Flash-guard raced on purpose: triggered `saveDraft()`'s flash, then called
  `updateWordCount()` mid-flash with different content - flash text survived
  untouched, and the eventual restore correctly returned to the
  *pre-flash* count (not the mid-flash one), with the next natural call
  catching up. Same pattern confirmed for the proofreader flash site.
- Re-ran tap-to-open, drag-to-switch, and the unrelated scene-status-arm
  drag after this round's `getRailBand()` rewrite - all still pass.
- **Desktop parity hit a real testing pitfall worth recording**: this
  browser environment's `navigate` does not always perform a true hard
  reload for `file://` URLs (confirmed even with a cache-busting query
  param) - DOM mutations from an earlier mobile-mode test round persisted
  across "reloads" in the same tab, which initially looked like a mobile-
  gate leak on desktop. Resolved with a self-contained test that doesn't
  depend on reload freshness at all: manually reset the element, ran a
  desktop pass (untouched), a mobile pass (correctly written), and a
  desktop pass again (still untouched, not overwriting the mobile value) -
  all three passed with `isMobileMode()` checked live at each step. Zoom
  control height (22px) and every other measured desktop value matched
  pristine `HEAD` exactly once verified this way.

## [2026-08-02] – Rail handle becomes the layout anchor, full drag range

Supersedes the drag-clamp fix earlier the same day (`2bf9b81`). That round
stopped the handle being dragged somewhere the hidden strip couldn't legally
expand into — a real fix, but it worked by *restricting where the handle
could go*. Aaron's follow-up ask inverts the relationship: the handle should
be draggable across the **entire** header-to-nav band, and the rail should
build itself around wherever the handle ends up, rather than the handle
being boxed in by where the rail can fit.

### Changed
- **The handle is now the single source of truth for the rail's vertical
  position**, not the rail. `layoutRailAroundHandle(rail, handleTop)`
  replaces the old `clampTop()` + centered-rail model: it clamps the
  *handle* into the full header-to-nav band, then positions the strip
  around that handle position (centering the strip on the handle where
  there's room, then clamping the **strip** — not the handle — into the
  band at the edges). Both `toggleGuidanceRailCollapsed()` and the drag
  handlers (`armDrag` / `onPointerMove`) now go through this one function
  instead of each keeping their own positioning logic.
- **Opening/collapsing the rail preserves the handle's Y position and does
  not re-center it on the strip.** Previously the rail's CSS `top: 50%`
  plus the handle's own `top: 50%; transform: translateY(-50%)` meant
  expanding a collapsed (and dragged) handle re-centered everything;
  now `toggleGuidanceRailCollapsed()` reads the handle's real position
  first and rebuilds the layout around it, so the six icons appear
  "immediately behind wherever the handle is parked," per Aaron's wording.
- **The drag range is now the full band, flush at both ends** — "the top
  of the nav button to the bottom of the header," per Aaron, with the rail
  opening entirely within those two boundaries and never over- or
  under-lapping either one. The previous round's 8px inset at each edge is
  gone; this round's spec is exact flush contact, not a safety margin.

### Fixed
- The handle-offset clamp (inside `layoutRailAroundHandle`, for the
  expanded case) initially bounded the handle to `[0, railH - 2×borderTop -
  handleH]` on the assumption it should never leave the rail's padding box.
  That's wrong by exactly one border-width at both extremes, because CSS
  resolves an absolutely-positioned child's `top` against the parent's
  **padding** box, not its border box — so reaching the rail's true outer
  edge legitimately requires an offset of `-borderTop`, one border-width
  *outside* the padding box. Measured as a systematic 2px shortfall against
  the band edges (36px requested → 38px rendered, and the same 2px short at
  the bottom). Fixed by moving both clamp bounds out by one `borderTop`:
  `[-borderTop, railH - handleH - borderTop]`.

### Verification notes
Measured at 440×956 with a seeded scene, `getBoundingClientRect()` against
the live `--mobile-header-h` / `--mobile-nav-clear` band (36–878px this
round), real `PointerEvent` sequences (`pointerType: 'touch'`):
- Handle Y preserved exactly across collapse→expand→collapse round trips at
  a mid-band park point, confirmed genuinely off-center on the resulting
  strip (not incidentally centered).
- Both band extremes land flush to the pixel (36/36 top, 878/878 bottom)
  in both collapse states, with the expanded rail's own top/bottom staying
  within the band in every case — no over- or under-lap.
- Direct drag (not via the collapse toggle) verified in both states: drag
  while expanded moves the whole rail with the handle; drag while collapsed
  stays collapsed and doesn't pop open.
- Re-ran the full existing gesture suite to confirm no regression: tap-to-
  open, drag-to-switch-drawers (rail stays put, switches correctly), the
  text-selection guard (`body.ui-dragging`) still arms/clears correctly
  during a handle drag, and the guidance drawer still clears the handle on
  open (`2bf9b81`'s fix, unaffected by this round's positioning rewrite).
  The unrelated scene-status circle drag was also re-checked and is
  unaffected.
- `resetGuidanceRailPosition()` updated to also clear the handle's own
  inline offset (it now carries one independently of the rail) — verified
  it returns to the pure-CSS default (rail centered, handle centered on
  rail) rather than leaving a stale offset from a prior drag.
- Desktop A/B against pristine `HEAD` at 1280×800: identical on every
  value checked (rail/toggle `display:none`, header/wordmark unchanged,
  `--rail-tab-w` reading unset) — this round only touched JS positioning
  logic and made no CSS changes, so desktop parity was never at risk, but
  re-verified per the project's standing rule anyway.

## [2026-08-02] – Suppress text selection while dragging UI elements

### Fixed
- **Grabbing the scene-status circle or the rail handle started selecting
  text** (Aaron, on device). Root cause is a gesture collision, not a styling
  oversight: both drags arm on a **300ms press-and-hold**, which is also iOS
  Safari's own trigger for select-word and the "Copy | Look Up" callout. The
  controls look like graphics but are real text as far as the selection
  engine is concerned — the status circle renders its status code (`UF` /
  `C` / `R`), and the rail tabs and handle are FontAwesome glyphs, i.e. font
  characters. So the hold selected them, and the selection then extended as
  the finger moved.
  Fixed in two layers:
  1. `user-select: none` + **`-webkit-touch-callout: none`** on
     `#scene-status-bar` and `#mobile-guidance-rail` (the latter covers the
     six tabs *and* the handle, which is a child of it), so a selection can
     never start on these controls. `-webkit-touch-callout` is a separate
     property — `user-select` alone does **not** suppress the iOS callout
     bubble.
  2. A `body.ui-dragging` guard applied only for the **duration of an armed
     drag**, since once a drag is running the finger travels over the editor
     and other page text that could anchor a selection en route. Both drag
     setups raise and clear it, including on `pointercancel`, which iOS fires
     when a gesture is interrupted.
  Deliberately **not** a permanent global `user-select: none`: copying scene
  text is a real thing Erica does, so the editor and guidance text stay
  selectable at rest. `clearSelectionRanges()` drops any selection that got
  established before the guard went up, but explicitly **skips a selection
  anchored inside `#editor`** — clearing that would fight her caret while
  she's writing.

### Verification notes
- At rest: status bar, status circle, rail strip, rail tabs, and rail handle
  all compute `user-select: none`; `#editor` still computes `auto`; no guard
  class on `body`.
- Guard lifecycle asserted across every path: absent before the hold arms,
  present once armed, absent after release — for the status-arm drag, the
  rail reposition drag, **and** the drawer-scrub path (which arms with no
  hold delay). Also cleared correctly on `pointercancel`, and never left
  stuck by a plain tap that never became a drag.
- `#editor` computes `none` *during* an armed drag and returns to `auto`
  after release.
- **Editor selection survives a drag**: seeded a 48-character selection
  inside `#editor`, ran a full rail drag, and measured 48 characters still
  selected throughout — `clearSelectionRanges()`'s editor exemption works.
- **Limitation, stated rather than glossed:** `-webkit-touch-callout` is
  WebKit-only, and Chromium drops unsupported properties at parse time, so it
  is absent from the CSSOM here and **cannot be verified in this
  environment** — confirmed present in the shipped source (three
  declarations) instead. Whether the iOS callout is actually gone needs
  Aaron's device. The `user-select` half is fully verified above.

## [2026-08-02] – Rail drag clamp fix, narrower rail, drawer clears the handle

Third mobile-only round the same day, on top of the handle reshape above.
Desktop A/B-measured against pristine `HEAD` at 1280×800, identical on every
value including the three new custom properties (all mobile-scoped, so they
read as unset on desktop).

### Fixed
- **The rail's press-and-hold drag could park the collapsed handle somewhere
  the full 6-icon strip couldn't fit.** `clampTop()`'s bottom/top bounds used
  `rail.offsetHeight`, which is only the 96px handle while collapsed — far
  shorter than the ~340px full strip — so the handle could be dragged lower
  (or higher) than the hidden icons would actually allow, and expanding the
  rail back out would push it off-screen. Fixed by always clamping against
  the **expanded** height, computed from the tab count and `--rail-tab-h`
  rather than measured live, so it works correctly precisely when the real
  strip isn't there to measure. First attempt undershot by 4px (874px bottom
  against an 870px legal limit) because it read the rail's own *current*
  border via `getComputedStyle`, which is `none` while collapsed — switched
  to reading `--rail-border-w` directly, which reflects what the border will
  be once expanded regardless of current state. Caught by direct measurement
  before shipping, not assumed correct from the code.

### Changed
- **Rail tabs are no longer square.** Aaron asked for the icon's top,
  bottom, and left margins to match (right stays the 8.8px set earlier the
  same day — not part of this ask). Height stays 56px, where the common-case
  20.8px icon already centers to a ~17.6px top/bottom margin; matching left
  to that only needs a 47.2px-wide box (17.6 + 20.8 + 8.8), instead of the
  old 56px square with ~26px of unused space on the icon's editor-facing
  side. **Removing that dead space is also what makes the rail narrower** —
  the two parts of the ask are the same change, not two separate ones.
  Padding is now explicit on both sides (`0 0.55rem 0 1.1rem`) rather than
  left-padding-plus-centering-slack, so the common-case icon's content-box
  exactly equals its own width and there's no residual slack for
  `justify-content: center` to redistribute unpredictably.
- **The scene-context (guidance) drawer opens further left**, clearing the
  handle instead of overlapping it. Its `right` offset was a flat `3rem`
  (48px) — enough to clear the old strip-foot toggle, not enough once the
  handle started protruding further left than the strip itself. Now computed
  as `--rail-tab-w + --rail-border-w + --rail-handle-w + 8px` (measured 10px
  final gap) instead of a re-guessed literal.

### Added
- **Three shared custom properties** — `--rail-tab-w`, `--rail-tab-h`,
  `--rail-border-w`, `--rail-handle-w` — read by the rail tabs, the rail's
  own border, the handle, the drawer's offset calc, *and* the JS drag clamp.
  This is deliberately the fix for a pattern that has now bitten the project
  twice (`--arm-pad-r`, CHANGELOG 2026-07-27; the folder-tab z-index note,
  earlier today): a position or size that depends on another element's
  geometry, expressed as a hardcoded literal, silently drifts the next time
  that geometry changes. Grep for these four names before resizing the rail,
  its border, or the handle again.

### Verification notes
- Margins on the common-case icon measured 17.1 / 18.1 / 17.6 / 8.8px
  (top/bottom/left/right) — small sub-2px variance from the FA glyph's own
  baseline metrics, same order of variance already accepted elsewhere in this
  branch. Rail's outer width 58px → 49.2px. All six tabs still touching
  (no gap introduced between them).
- Drawer-vs-handle clearance re-measured after the width change: 10px gap,
  `drawer.right <= handle.left` holds.
- Drag clamp re-tested by dragging the collapsed handle *past* both legal
  edges (to y=2000 and y=-500 on a 956px-tall viewport) and confirming the
  clamp caught it before expanding, then expanding and re-measuring: bottom
  landed at exactly 870px (matching the computed legal limit `870`), top at
  exactly 44px (`--mobile-header-h` + 8). Both directions.
- Gestures re-run after the resize: tap-to-open, drag-to-switch (switches,
  rail does not move), and press-and-hold reposition from a rail icon (moves,
  drawer does not open) — all still pass at the new 47.2px tab width.
- Touch target re-checked: 47.2×56 still clears the 44px primary-tier floor.
  Icon contrast unchanged at 6.39:1.

## [2026-08-02] – Wordmark, rail edge/margin, and the rail folder tab

Four more mobile-only adjustments at Aaron's direction, same day as the icon
sizing round below. Desktop A/B-measured against the pristine pre-change file
at 1280×800 and identical on every value.

### Changed
- **"Big Tiff" wordmark sized to the feather glyph** (30px → 15.2px), not to
  the gold disc it sits in — the disc is 30px but the feather inside it is
  half that. Both now read from one new `--logo-glyph` custom property rather
  than repeating the literal, since "the wordmark matches the feather" is
  exactly the kind of relationship that breaks silently when only one of two
  rules gets edited (see the `--arm-pad-r` bug, 2026-07-27).
  **Side effect worth knowing:** the mobile header was wrapping to two rows to
  fit the 30px wordmark. It now fits on one, so `--mobile-header-h` measures
  **70px → 36px** and everything anchored to it (the status arm, the sheets)
  moves up with it. That is a real layout shift, not just a font change — it
  gives back 34px of writing area.
- **Rail's outer gold edge 1px → 2px.** Width only; the 0.55 alpha and the
  1px inter-tab dividers are untouched (the ask was the outer edge).
- **Rail icons moved closer to the screen edge**, gap halved 17.6px → 8.8px.
  Done as `padding-left` on the 56px tab rather than right-aligning the
  glyphs, so the six icons keep a shared centerline — they have different
  intrinsic widths (mountain-sun 26px vs bullseye 20.8px) and flex-end would
  have left their left edges ragged. Touch targets stay 56×56.
- **The rail's collapse control is now a handle on the rail's long left
  wall**, replacing the 56×30 arrow at the foot of the strip. Vertically
  centered, protruding into the editor. Collapsed, the six-icon strip
  disappears and the handle *becomes* the rail — it drops back into normal
  flow so the rail element sizes to it, which matters because the drag clamp
  reads `rail.offsetHeight`.
  Built first at 60×140 (2.5 rail icons tall, twice the old toggle's depth),
  then **revised the same day to 24×96** after Aaron pointed at iOS's own
  hidden-PiP-window handle and asked for that size and shape: ~55×217 px in a
  923px-wide screenshot of a 393pt screen (≈2.35×) → ~24×96 CSS px, 4:1, with
  a fully rounded outboard edge (12px radius = half the width). Chevron
  dropped 0.85rem → 0.7rem to sit inside the narrower pill.
  **24px wide is below the project's 30×30 chrome touch floor** — a
  deliberate, Aaron-specified exception now recorded in
  `BUTTON_STYLE_GUIDE.md`, on the reasoning that the control is edge-anchored
  and 96px tall. Flagged to him rather than silently adjusted.

### Added
- **`.rail-tab-strip`**, an inner scroller wrapping the six tabs. Required,
  not cosmetic: the rail must be `overflow: visible` for the folder tab to
  protrude, and CSS will not allow `overflow-x: visible` alongside
  `overflow-y: auto` (the visible axis computes to auto). The scroll +
  max-height behavior that kept tabs reachable at large text scales moved
  into this wrapper unchanged.

### Fixed
- **Folder tab was unreachable under an open drawer.** Protruding 60px left
  put it inside the drawer's footprint, where the old strip-foot toggle never
  went. Raising the *tab's* z-index to 470 did nothing — the rail is
  `position: fixed` with a z-index, so it forms a stacking context and no
  child z-index can escape it. Fixed on the rail itself: **45 → 465**.
  Deliberately not 470: `#scene-status-bar` is 470 and the rail comes later in
  the DOM, so an equal value would silently invert Aaron's documented "the
  status circle wins that overlap" rule. 465 clears the drawer (460) and stays
  under the arm.
- `.rail-tab:nth-last-child(2)` → `.rail-tab-strip .rail-tab:last-child`. The
  old selector meant "the tab above the toggle"; with the toggle out of the
  flow it would have skipped the real last tab and stranded a divider on the
  strip's bottom edge.

### Verification notes
Measured at 440×956 with a seeded scene; gestures as real `PointerEvent`
sequences (`pointerType: 'touch'`):
- Wordmark and feather rects both 15.2px tall with matching tops — "same
  height" true of the measured rect, not just the type size. Branding not
  clipped, account button still fully on-screen at the new header height.
- Icon gap 8.8px on all four 20.8px glyphs (exactly half of 17.6px); the two
  wider glyphs land at 6.2px and 7.5px, the same per-icon variance the
  centered layout already had. Tabs still 56×56, `box-sizing: border-box`.
- Handle measured 24×96 (ratio exactly 4.00), vertically centered on the rail
  to within 0.5px, 2px border on three sides, `12px 0 0 12px` radius, chevron
  fitting inside with clearance, not clipped. Collapsed: strip `display:none`,
  handle `position:static`, rail 24×96 flush right, rail border/background
  cleared so the edge isn't doubled, chevron rotated, `aria-expanded` false.
  Re-expanding restores every value. (The intermediate 60×140 version was
  measured the same way before the revision.)
- Gestures re-run after the DOM restructure: tap-to-open, drag-to-switch
  (switches, rail does not move, trailing click swallowed), press-and-hold
  reposition **from a rail icon and from the folder tab itself** (drags
  without collapsing), tap-to-collapse/expand, and `resetGuidanceRailPosition`.
- Stacking verified by `elementFromPoint`, not by reading z-index values: the
  folder tab and the rail icons are the topmost elements at their own centers
  with a drawer open, and collapse works from under it. The status arm was
  then parked in a real geometric overlap with the rail and still came out
  topmost — Aaron's spec holds.
- Accessibility: rail tabs 56×56, wordmark 15.2px at 5.44:1, rail icons
  6.39:1. The handle's 24px width is the one known floor exception, above.
- Desktop A/B at 1280×800 against pristine `HEAD`: header 1280×58, wordmark
  still 21px/31.5px, feather 16px, gold disc 32×32, STORYFORGE and account
  button at identical offsets, rail and toggle `display:none`, rail icon still
  16px with 0 padding, `--logo-glyph` unset. The new `.rail-tab-strip` div
  exists in the DOM there but renders nothing.

### Flagged, not settled
- The header is a **shared** element and Aaron has said it likely needs
  splitting into separate mobile/desktop rules. The wordmark change above is
  mobile-gated, so nothing desktop moved — but that split is still an open
  idea he has NOT approved. Don't action it unprompted.

## [2026-08-02] – Icon sizing round: rail +30%, action bank +30%, DL code

Three small mobile-only adjustments at Aaron's direction. Desktop re-verified
unchanged by direct measurement against the pristine pre-change file, same
discipline as every prior round.

### Changed
- **Guidance-rail tab icons +30%** (`1rem` → `1.3rem`, measured 16px →
  20.8px). Box, padding, and the 1px divider borders all unchanged — this is
  a glyph-only change, so the 56px touch targets and the strip's connected
  geometry are untouched. **This settles the "too much margin around the rail
  icons" item that had been open since 2026-07-27**, using Aaron's own
  number; the earlier reverted attempt (2.5rem, unasked) overshot badly, and
  1.3rem lands nowhere near it.
- **Scene-status action-bank icons +30%** (inherited `0.9375rem` →
  `1.21875rem`, measured 15px → 19.5px) on Sammy / Draft Log / Save Draft.
  Aaron's note said "button size"; confirmed with him first that the **glyph**
  grows and the box does not. Growing the boxes 30% would have taken them to
  57.2px — wider than the 56px status circle above them — which would widen
  the whole arm and invert the "narrower buttons centered under the wider
  circle" relationship that the arm's 8px/14px padding values were derived
  from. Boxes stay 44px `--tap`; arm width stays 74px.
- **Draft Log shows the letters `DL` instead of its icon, mobile only.** Uses
  the existing `.status-trigger-label` / `.status-trigger-code` pattern —
  both forms in the DOM, CSS picks per layout, no JS branching. Desktop still
  renders the `fa-layer-group` icon plus the full "Draft Log" text, and the
  `aria-label` is unchanged either way, so the accessible name is untouched.
- **Collapsed arm's bottom green trimmed.** `padding-bottom` drops 14px → 0
  **in the collapsed state only**; the open state keeps its 14px. That 14px
  was derived from the open state (matching the 14-16px side clearance the
  centered action buttons get) and made no sense with the buttons hidden — it
  stacked on top of the chevron toggle's own ~9.4px internal bottom padding,
  leaving ~25px of green under the glyph against only 8px above the circle.
  Now the toggle's own padding is the entire bottom gap: measured 9.4px green
  plus the 2px gold border, which reads symmetric with the top.

### Verification notes
Measured via `setMobilePreview(true)` at 440×956 with a seeded scene, real
`getBoundingClientRect()` / `getComputedStyle()` values, not screenshots:
- Both icon bumps confirmed at exactly ×1.3 (16→20.8px, 15→19.5px) with
  boxes, padding, and borders byte-identical to before (56×56 rail tabs,
  44×44 action buttons, arm still 74px wide, circle still 56px).
- Collapsed/open padding swap asserted in both directions with the state
  class read explicitly (`actions-open`): collapsed → 0px pad / 96px arm /
  9.4px green under the glyph; open → 14px pad / 250px arm / 16px under the
  last button, i.e. the open state is untouched. `#auto-save-status` stays
  visible in both states (the documented must-not-hide invariant).
- Gestures re-run as real `PointerEvent` sequences (`pointerType: 'touch'`):
  status-arm press-and-hold drag → corner-dock left and right (flush, correct
  border/radius morph per side, `top` still pinned to the header), status menu
  direction-flip measured at a 10px gap and fully on-screen in **both**
  directions, rail tap-to-open, rail drag-to-switch (switches, rail does not
  move, trailing click swallowed), and rail press-and-hold reposition (moved
  the full 130px, drawer correctly did not open, reset clears the inline
  style).
- Accessibility re-checked on everything touched: rail-tab targets still
  56×56, action buttons still 44×44, rail icon contrast 9.99:1, `DL` contrast
  5.44:1 at 19.5px — all above the AA floors in `BUTTON_STYLE_GUIDE.md`.
- **Desktop verified by A/B measurement**, not assertion: the pristine
  `HEAD` file and the edited file were loaded side by side at 1280×800 and
  probed identically. Every value matches — status bar `1028×48` at the same
  offset, same padding/border/radius/z-index, status pill 100×18.55 at
  10.4px, action buttons 95.2 / 90.86 / 80.67px wide with 12px icons and
  visible text labels, rail `display:none`, rail icon still 16px there. The
  new `.btn-code` span computes to `display:none` / 0×0 on desktop, so Draft
  Log's width is unchanged to the hundredth of a pixel.

### Flagged, not settled
- The collapsed-arm bottom gap is now 9.4px because that is the chevron
  toggle's own internal padding, which is the floor without touching the
  toggle's box (deliberately left alone — it carries a documented "don't
  re-fix without re-measuring" note about the 30px touch-target floor). If
  Aaron wants it tighter than 9.4px, that toggle's padding is the next thing
  to change, and it needs asking first.
- No real-device confirmation yet for this round specifically.

## [2026-07-27] – Scene-status arm: drag, corner-morph docking, button polish

Four rounds in one day on the mobile scene-status arm (the circular R/UF/C
control up near the header): a button-sizing/spacing polish pass, a new
press-and-hold horizontal drag, corner-morph docking for that drag, and a
same-day fix-up round after Aaron reviewed the corner-morph round and flagged
real problems. All mobile-only; desktop re-verified unchanged after every
commit, same discipline as every prior round.

### Added
- **Horizontal drag for the status arm** (`setupStatusBarDrag()`), mirroring
  the existing guidance-rail drag: a 300ms press-and-hold on the status
  circle arms dragging; a plain tap still opens the status options. The
  arm's `top` stays pinned to the header; only `left` moves, clamped to the
  screen edges. Position is inline-style-only (never persisted) and resets
  on login, same as the rail.
- **Corner-morph docking.** Release the arm within **48px (0.5in)** of a
  screen edge and it snaps flush and morphs: the wall-facing border and that
  bottom corner's rounding drop, the free side keeps both (both bottom
  corners round when the arm is genuinely free-floating mid-header).
  `updateStatusBarDock()` runs on drag release, every scene render, and
  login reset, so the shape is always correct for wherever the arm actually
  is — not just right after a drag.
- **Direction-aware status-option menu.** The circle's slide-out option menu
  (Unfinished/Complete/Review) now opens left instead of right when there
  isn't room, decided fresh at open time (`positionStatusMenuDirection()`)
  from the option row's real `scrollWidth` against actual viewport space —
  not a static docked/floating flag, since a floating (undocked) arm near
  the right edge needs the same flip a docked one does.
- **`BUTTON_STYLE_GUIDE.md`** — a shared reference for button size tiers,
  icon-size ranges, and accessibility minimums (contrast, touch targets)
  across both desktop and mobile layouts, added after the rail-icon sizing
  went back and forth twice in one day with no written reference to check
  against.

### Changed
- Scene-status arm's gold border now wraps left/right/bottom (previously
  missing the left edge); stays borderless on top, flush under the header,
  regardless of horizontal position.
- Margins standardized to 8px throughout the arm (was an inconsistent
  4/4/4/2px) — container padding and the circle-to-chevron gap. Bottom
  padding under the action-button stack specifically bumped to 14px to
  match the ~14-16px side clearance the buttons actually get once centered
  in the wider, circle-driven column (the bottom edge has no equivalent
  centering slack, so flat 8px there visibly under-matched the sides).
- Status-actions-bank buttons (Sammy/Draft Log/Save) now centered under the
  arm instead of flush-left.
- Guidance-rail collapse toggle moved from the top to the bottom of the
  six-tab bank, so it never ends up under the now-draggable status arm.
- Status circle's own gold ring thinned 2px → 1px.
- Scene-summary/review-note boxes: removed a stale fixed 6.375rem left
  indent that only made sense back when the arm was permanently pinned
  flush-left; replaced with a plain 16px safe margin now that the arm can
  dock anywhere or float free (and, being a high-z-index overlay, may sit
  on top of the text when parked over it — an accepted trade-off, same one
  the guidance rail already makes on the right).
- Show-more/less chevron (`.m-expand`) in the Scene Summary and Review Note
  boxes centered horizontally (was left-aligned); those boxes' bottom
  padding trimmed 12px → 4px since the chevron row already carries its own
  vertical buffer.

### Fixed
- **Status-option menu opened from the wrong position entirely** —
  `left: calc(100% + var(--arm-pad-r) + 2px)` referenced a custom property
  (`--arm-pad-r`) that had been deleted earlier the same day when the arm's
  padding was standardized to a flat 8px, but this one declaration still
  referenced it. An undefined custom property inside `calc()` invalidates
  the *whole* declaration (not just falls back to a default), so `left`
  silently reverted to the desktop rule's `left: 0` — the menu rendered
  overlapping the circle's top-left instead of sliding out beside it.
  Caught by Aaron on the live preview, not by the synthetic-event test
  suite, since the tests asserted state transitions (open/closed, direction
  class) rather than the actual rendered pixel position — worth adding
  position assertions to this kind of test going forward.
- Off-by-2px right-dock snap: docking right removes that side's border,
  shrinking the box *after* the snap position had already been calculated
  from the wider (still-bordered) box. `updateStatusBarDock()` now
  recomputes the target position after applying the dock class.

### Reverted
- **Rail-tab icon size, back to 1rem** (had been grown to 2.5rem earlier
  the same day). The original ask was "too much margin around the icons" —
  not a request to enlarge the glyph. Growing it read as oversized and
  broke consistency with the editor-toolbar/status-button icon sizing
  established the same day. No replacement fix shipped; left open rather
  than guess again. **Flagged, not settled** — if the margin still needs
  addressing, the fix has to come from something other than glyph size.

### Verification notes
Every state change verified by direct measurement (`getBoundingClientRect`
diffs, `getComputedStyle`), not by eyeballing screenshots, via
`setMobilePreview(true)` + synthetic `PointerEvent` sequences
(`pointerType: 'touch'`), consistent with the project's established test
pattern:
- Corner-dock: measured in all three states (free-floating — both bottom
  corners round, full 2px border both sides; docked-left; docked-right —
  mirrored border/radius each time), plus the 48px/24px threshold boundary
  (drag-and-release at 40px docks, at 60px doesn't).
- Direction-flip menu: measured the default (right) open position after
  the `--arm-pad-r` fix (10px gap, vertically aligned with the circle, not
  overlapping it), then dragged the arm to the right wall and confirmed the
  menu opens left with the same 10px gap and stays fully on-screen.
- Bottom-margin-matches-side-margin: measured the actual visually-last
  button (CSS `order` reorders Sammy/Draft Log/Save, so DOM-last ≠
  visually-last — used `getBoundingClientRect().bottom` comparison, not
  DOM position, to find the real bottom button).
- Desktop re-checked unchanged after every commit (status pill still drops
  straight down from the trigger, same width, no mobile-only classes
  applied without `body.mobile-layout`).

### Flagged, not settled
- Rail-tab icon margin (see Reverted above) — original complaint
  unresolved, needs a fix that doesn't touch icon size.
- No real-device (iPhone) confirmation specifically for this round's new
  gestures (arm drag, corner-dock snap) yet — see
  `Mobile_Port_Handoff.md`'s real-device-testing section for the general
  (corrected 2026-07-27) testing-status context.

## [2026-07-26] – Achievements & Usage Tracking (data layer)

The engine behind 188 achievements plus the usage tracking that drives them.
**No new UI yet** — the achievement book, toasts, icons, and creature
animations are a separate later phase. The Stats window's existing 6-badge
teaser now shows real data instead of the old hardcoded mock.

### Added
- **`progress.json`** — a new, **AES-encrypted** file in the data repo
  holding all behavioural data: counters, per-day buckets, hour-of-day
  histogram, personal bests, per-scene records, streaks, unlock log, and
  weekly state. Same envelope shape as `state.json`, so it reuses the
  existing encrypt/push/hydrate pattern.
- **188 achievement definitions embedded inline** (~31 KB), generated from
  `storyforge-achievements.json` v3.1.0 and cross-checked against the source
  (0 mismatches on ids, titles, and every word threshold).
- **Named-event tracking at ~25 call sites** — one line each
  (`trackEvent('libraryOpen')`). Covers panel opens, notepad, draft pad,
  snapshots create/restore, focus modes, proofreader run + apply, Sammy
  sends, canon syncs, changelog, library search, guidance cards, mobile
  drawers, scene selection, and status changes. **Not** a generic click log,
  by choice.
- **Writing sessions** (a genuinely new concept — none existed). 15-minute
  idle timeout, ends on idle/unload/midnight, and *resumes* across a page
  refresh so a 1,900-word session doesn't lose its shot at the 2,000-word
  badge. Deliberately starts on **any** interaction, not just typing, so the
  "zero-word session" achievements can actually fire.
- **In-memory word index**, replacing the full-localStorage rescan that
  `getStatsData()` and `syncStatsToRepo()` ran on every call — that was
  O(scenes × draft length) and slow on a phone with 100+ scenes.
- **`getAchievementBook()`** — returns all 188 with unlocked state, percent,
  remaining, and a formatted label, plus hidden-achievement masking.
  Everything the later book UI needs, measured at 1.8 ms.

### Changed
- **`stats.json` bumped to schemaVersion 2.** It no longer carries a
  2,000-entry plaintext activity log. That log published a running record of
  *when Erica writes* to a **public** repo — a real privacy leak that
  predates this work. It is now drained on the first v2 write, and that
  detail lives encrypted in `progress.json` instead. The dead
  `achievementsUnlocked` array is replaced by an
  `achievementsUnlockedCount` scalar, so a viewer profile can render
  "13 / 188" without decrypting anything or leaking *which* badges.
- **Stats window** now shows the real unlock count out of 188, and its six
  tiles are drawn from actual engine state (unlocked first, then whatever is
  closest to unlocking) rather than the previous hardcoded list, three of
  whose entries were permanently `false`.

### Fixed
- **`initMobileMode` only wired `editor.blur`** despite the comment above
  `mobileImmediateSave` promising blur/hide/pagehide. Backgrounding Safari
  without blurring the editor skipped the immediate draft push. Now also
  wired to `visibilitychange` and `pagehide`.

### Design notes worth knowing
- **Write amplification** was the main risk, since every push is a git
  commit. Local-first: every mutation hits localStorage immediately; the
  repo push is backup only, on a 90 s idle timer behind a hard 5-minute
  floor. Measured: **120 typing events over 12 s produced exactly 1
  progress commit.** That's quieter than the existing draft push, which
  already commits every 2.5 s of typing pause.
- **The merge is a join-semilattice** — every field merges by max, earliest,
  or union, so two devices reconcile to the same result in either direction.
  Verified commutative, associative, and idempotent.
- **The analytics layer can never break the editor.** Every entry point is
  wrapped; three failures trip a kill switch for the session. Verified by
  sabotaging the layer mid-session and confirming typing, autosave, word
  count, and all other localStorage keys were unaffected.

### Resolved after review
- **`full-outline-clear` is now human-confirmed** (Aaron, 2026-07-26). Its
  original trigger ("every scene has a terminal status") read literally
  counted `unfinished` — the *natural resting state* of a scene — as
  terminal, so it unlocked on a brand-new project having done nothing.
  Settled: **no automatic reading of the outline can prove the book is
  finished; only Erica saying so can.** The engine's only job is deciding
  when it's fair to *ask* — every scene Complete, nothing flagged for
  review — and then it prompts. Declining leaves it locked. It asks at most
  once per day, so a last scene that wobbles in and out of Complete can
  never nag.

### Flagged, not settled
- **`manuscript-assembled` is currently unreachable** — it targets a
  full-draft-assembly feature that doesn't exist. The `counter.assemblyRun`
  metric ships at 0 so it works the day that feature lands.
- **Weekly system is deferred** pending the full pool (the JSON has 13
  entries; Aaron is sourcing more). The selection logic is pool-size
  agnostic, so adding them is a data edit with no code change.

### Verification notes
Boundary thresholds asserted exactly (10,000 words fires `words-10000` but
not `words-11000`; 9,999 fires neither; 3rd distinct day; 0.75 ratio but not
0.74). Streaks verified including the day-3-vs-day-4 `streak-keeper`
boundary and stale-streak reset. No double-unlock on repeat sweeps. Merge
verified commutative/associative/idempotent with an order-insensitive
comparison. Full 188-rule sweep measured at **0.1 ms**, dirty-subset sweep
0.05 ms. Session resume across reload confirmed with no double-counted
session. Mobile path exercised via `setMobilePreview(true)` at 440×956.
Desktop re-verified unchanged: 100×19 pill at `9999px` radius showing the
full word, arrow tab hidden, action bank always visible, copy buttons
hidden, status bar `sticky`, guidance rail hidden. **0** console errors
throughout.

`full-outline-clear` gate tested across all five states: all-unfinished
(not eligible, no prompt), partially complete (not eligible), all complete
*with* a review flag (not eligible, no prompt), all complete with flags
cleared (eligible, prompts once), declined (stays locked, fact stays 0,
no nag on same-day re-check), and confirmed on a later day (unlocks, and
the confirmation survives a merge in both directions). Fresh-project
regression re-checked: still 0.

## [2026-07-25] – Guidance Copy, Rail Drag, Circular Status (sixth review)

Four requests: a copy button on each guidance drawer, larger guidance-rail
icons, a draggable rail, and a full redesign of the mobile status control
from a pill to a circle. All mobile-only; desktop untouched.

### Added
- **Copy-to-clipboard button** (`.guidance-copy-btn`) in each of the six
  guidance drawers (Purpose/Canon/Emotional/Beats/Setting/Success), next to
  where the desktop-only minimize/expand icons would be. Reads the drawer's
  already-rendered text via `innerText` (so list formatting matches what's
  on screen) and writes it with `navigator.clipboard.writeText()`. The icon
  flashes to a checkmark for 1.2s on success (`copyGuidanceCardText()` /
  `flashGuidanceCopyButton()`).
- **Guidance rail drag-to-reposition.** A 1-second press-and-hold on the
  rail arms dragging (visible gold glow cue via `.rail-dragging`); a
  subsequent drag moves the whole rail up/down the right edge, clamped
  between the header and bottom-nav clearance
  (`setupGuidanceRailDrag()` in an IIFE near `updateRailActiveState`). A
  plain tap under 1s still opens a drawer as before — the drag-arming click
  is swallowed via a capturing click listener keyed on whether a drag
  actually happened. Position is **never persisted** (no localStorage) — it
  lives only in an inline `top` style on the live element, so a fresh page
  load (a real login) always starts centered with zero extra reset code.
  `submitLogin()` also calls `resetGuidanceRailPosition()` explicitly, for
  the case where a session is already loaded and a different account logs
  in without a full page reload.

### Changed
- **Guidance rail icons enlarged** from 44×44px to 56×56px (icon glyph
  1rem → 1.375rem), staying a single touching "connected bank" (one shared
  strip border, no per-icon borders) rather than becoming separate spaced
  buttons like the Sammy/Draft Log/Save bank. **Flagged, not fully
  settled:** the rail icons and the action-bank buttons were already
  pixel-identical at 44×44 before this change — Aaron's brief presupposed a
  size gap that didn't exist in code. Asked which axis he actually meant
  (bump size, add per-icon borders, or match the old 88px pill width) and
  got no reply before continuing, so this went with the plainest reading —
  make them visibly bigger, keep the connected layout — as a placeholder.
  Revisit if 56px isn't what he had in mind.
- **Mobile status pill replaced with a circular status button.** The outer
  green-arm / 2px-gold-border shape (`#scene-status-bar`) is unchanged;
  only what sits inside it changed. The trigger is now a 56px circle with a
  1px gold border showing a single status code — **R** (Review), **UF**
  (Unfinished), **C** (Complete) — instead of the full word, sized to match
  the enlarged rail icons. `updateStatusSelector()` now renders both a
  `.status-trigger-label` (full word, shown on desktop) and a
  `.status-trigger-code` (short code, shown on mobile) per option, so
  desktop kept its original pill unchanged.
- **Status options slide out horizontally** instead of dropping down
  beneath the trigger (mobile only) — the circle sits flush against the
  left screen edge, so a horizontal reveal to the right made more sense
  than down-and-under. Same alternate-status logic as before (current
  status excluded, Review only offered if already active).
- **Sammy / Draft Log / Save now collapse behind a new dropdown-arrow tab**
  (`#status-actions-toggle`, `toggleStatusActionsBank()`), starting closed
  on every scene load. The three buttons were re-wrapped in their own
  `.status-actions-bank` div, separate from the row that also holds
  `#auto-save-status` — collapsing that row directly would have hidden the
  auto-save indicator too, since `display:none` on an ancestor hides a
  `position:fixed` descendant regardless of its own positioning. The arrow
  tab itself is 56×30px (chrome-tier `--tap-sm` floor, not a new
  below-floor size) — initially built it at 22px tall before catching that
  it violated the touch-target floor and fixing it in the same pass.

### Verification notes
All four checked at 440×956 via `setMobilePreview(true)` / synthetic
`PointerEvent` sequences (real touch hardware not available in this
environment — see the "not verified" list this doc already carries
forward from the original mobile port). Copy button: present in all six
drawers, click fires with no console errors, icon swaps to a checkmark.
Drag: long-press (1100ms) arms `.rail-dragging` and a following
`pointermove` repositions the rail by the exact delta; a short tap (150ms)
still opens the correct drawer and never arms dragging; an extreme drag
(+5000px) clamps to `railBottom < viewport height`, never off-screen;
`resetGuidanceRailPosition()` restores the exact original centered
position. Circular status: 56×56 circle, `border-radius: 50%`, correct
code per status (R/UF/C), full label present but `display:none`; clicking
opens a horizontal menu with only the valid alternate status, clicking it
changes `currentScene.status` and closes the menu; arrow tab opens/closes
the action bank without affecting `#auto-save-status`'s visibility, and
resets to closed on every new scene load. Full-open-state audit (drawer +
action bank + status menu all open at once): **0** interactive elements
under 30px anywhere in the scene view, no layout overlap between the
status arm, guidance rail, drawer, and status menu. Desktop re-verified
unchanged: pill still `border-radius: 9999px` showing the full word, arrow
tab and copy buttons both `display:none`, action bank always visible
(never collapses), rail hidden.

## [2026-07-25] – Control Sizing Corrections (fifth review)

### Fixed
- **Status pill dimensions were inverted in the previous entry.** "Height of
  the Scene Summary title line" had been applied as the pill's *height*
  (20px tall × 60px wide). Corrected to the actual intent: the pill is
  **double the Sammy button's width and the same height** — 88 × 44px.
  Its dropdown options follow the wider pill.

### Changed
- **Green control shape tightened**: padding reduced to a uniform 4px and the
  buttons left-aligned rather than centred, so the pill and all three action
  buttons sit the same short distance from the window edge with an equal
  green margin all round. The summary/review indent follows the wider shape.
- **List tool removed** from the editor toolbar (desktop and mobile — the
  request wasn't mobile-scoped), leaving B / I / U / Proofreader / zoom.
- **Glasses icon centred** within its button.
- **Show more/less chevrons now use the whole line as the hit area** (308px
  wide instead of 48px). The row is otherwise empty, so this costs no
  vertical space at all.
- **Auto-save text is now light brown (`#8B6F47`)** and pulled further in
  from the editor's rounded border so the two never touch.

### Verification notes
Measured at 440×956: pill 88×44 with Sammy at 44×44 (exactly double width,
equal height); all four controls at x=4 inside a 4px-padded shape; list
button absent; glasses `justify-content: center`; chevron hit area 308px
matching the summary's inner width; auto-save at `rgb(139,111,71)` ending
14px clear of the editor's right border. Toolbar still one line. **0**
sub-12px text, **0** controls under 44px, no horizontal scroll, no console
errors across eight views. Desktop re-measured unchanged: sticky 48px green
row, auto-save still inside it in the original cream, 100×19 pill, book icon
visible, 126px margins, 14px editor, 320px panels.

**Intentional sub-AA exception:** the auto-save line now measures **1.34:1**
— that is what "hardly noticeable" produces, and it was asked for
explicitly. It is decorative reassurance only; saving happens regardless and
is not conveyed by that text alone. Easy to raise if it turns out to matter.

## [2026-07-24] – Proofreader "Go To" + Header/Control Polish (fourth review)

Seven notes from Aaron's fourth pass. Six are visual; the seventh adds a
genuinely new Proofreader capability.

### Added
- **Proofreader "go to" arrow.** Every listed issue now carries an arrow that
  closes the Proofreader, scrolls the offending text into view, and
  highlights it. **It never mutates the draft**: wrapping the match in a
  `<span>` would land in `editor.innerHTML` and autosave would persist the
  highlight into Erica's prose. Instead it paints a `Range` via the CSS
  Custom Highlight API (`::highlight(proofreader-target)`) where available,
  and falls back to a plain text selection otherwise — both purely visual.
  Because `getEditorPlainText()` injects newlines at block boundaries, the
  issue offset doesn't map 1:1 onto DOM text nodes, so it picks the
  occurrence *nearest* the expected offset rather than the first match. If
  the draft changed and the text is gone, it says so instead of failing
  silently.

### Changed
1. **Header logo**: the circular mark now matches the Login button's height
   (30px on mobile), and the "Big Tiff" wordmark matches the circle.
2. **The pill corner and button arm are one shape** — a single green block
   with one 2px solid gold outline, instead of two separate green areas.
3. **Status pill is shorter but longer**: 20px tall (the Scene Summary title
   line height) and 60px wide, occupying the slot the summary's book icon
   used to hold — that icon is hidden on mobile since the pill now sits
   there. The morph-down options match the thinner pill exactly.
4. **Sammy / Draft Log / Save** reordered in the arm (Sammy on top).
5. **Scene summary and review note content moved left**, now sitting 21px
   clear of the green shape instead of indented well past it.
6. **All editor tools on one line** above the field, and the Proofreader's
   magnifying glass is now a pair of glasses. Measured that full 44px-wide
   tools still fit one line at 440px, so the single row costs nothing in
   touch size.

### Verification notes
Go-to arrow tested end to end: correct word highlighted for two different
issues, correct fallback selection when the Highlight API is stubbed out,
**draft `innerHTML` byte-identical before and after** on both paths, and the
"couldn't find that text" path when the draft no longer contains the match.
Full audit at 440×956 across scene / proofreader / notepad / stats / text
size / outline / library / landing: **0** sub-12px text, **0** WCAG AA
contrast failures, **0** controls under 44px, no horizontal scroll, no
console errors. Desktop re-measured unchanged: 32px logo mark, 21px
wordmark, sticky 48px green row bar with its 5px gold cap, book icon still
visible, 16px summary padding, autosave still inside the bar, 126px margins,
14px editor, 320px panels.

**Deliberate sub-44px exceptions (unchanged from the last round, both
requested):** the status pill (20px, item 3 asked for shorter) and its
dropdown options (30px). Everything else is at or above 44px.

## [2026-07-24] – Scene View Redesign: Green Control Arm (third device review)

Nine notes from Aaron's third pass. The biggest is a structural change to the
scene view: the horizontal green status bar becomes a fixed vertical control
arm in the upper-left, with the editor scrolling beneath it.

### Changed
1. **Outline and Library panels open to 70%** of the window width (was 50%).
2. **Status pill** is now the same height as the guidance rail buttons
   (44px), and its morph-down options match the thinner pill — inset by the
   green padding so the menu lines up with the pill instead of jutting past.
3. **"Auto-saved just now"** left the status bar and now sits semi-transparent
   in the bottom-right corner of the writing area, `pointer-events: none` so
   taps pass through to the editor beneath.
4. **The green status bar is now a fixed top-layer control arm** pinned under
   the header: the status pill sits in a green corner, and a green arm runs
   down the left edge with Save / Draft Log / Sammy stacked as 44px square
   icon buttons matching the guidance rail opposite. Editor content scrolls
   beneath it.
5. **The thick gold line** is now the scene summary's bottom border (it used
   to cap the status bar, which no longer sits there).
6. **Guidance rail buttons are square** (44×44).
7. **The rail is a top layer** and the editor now extends to the right edge
   with the same margin as the left, scrolling under the tabs.
8. **Show more/less is a bare chevron** on its own bottom line, lower-left —
   no text label, which reclaims a lot of vertical space.

### Fixed (found during verification, not reported)
- **Nav-height feedback loop.** Measuring the nav's real height into
  `--mobile-nav-h` fed straight back into its own buttons' `min-height`,
  so each measurement grew the nav (62→110px) and bottom-anchored elements
  drifted under it. Split into `--mobile-nav-h` (button height, fixed) and
  `--mobile-nav-measured` (runtime total incl. the Focus-bulge padding and
  safe area). Verified stable across repeated runs.
- **Summary/review text was permanently hidden** behind the new arm. Those
  two blocks are now indented clear of it; the editor still scrolls beneath
  as intended.
- **Review-note heading contrast, pre-existing:** gold on the box's
  translucent `bg-[#D4AF37]/15` measured **4.19:1**, under AA. Surfaced only
  after the contrast audit was corrected to composite an element's *own*
  translucent background (it had been scoring translucent chips against the
  wrong backdrop). Fixed with a solid `#3D2B1F` ground on mobile — same
  palette, same gold border — taking it to **6.4:1**.
- Guidance rail now scrolls rather than clipping if a large text scale makes
  six tabs outgrow a short landscape viewport.

### Verification notes
440×956, 956×440, and at 75–175% UI text scale: **0** text under 12px, **0**
WCAG AA contrast failures (with the corrected compositing), **0** primary
controls under 44px, no horizontal scroll, no console errors. Landscape
re-checked after the nav fix: arm, rail, sheets and the auto-save chip all
clear the nav. Desktop re-measured and unchanged: status bar still a sticky
48px green row, auto-save still inside it and static, expand arrows hidden,
review box keeps its `/15` tint, 320px panels, 126px margins, 14px editor,
100px pill, guidance columns side-by-side.

**Deliberate sub-44px exceptions, flagged not settled:** the status dropdown
*options* (~33px) and the expand chevrons (48×32) are both smaller than the
44px floor because item 3 asked for smaller options and item 8 asked to
reclaim vertical space. The chevrons were widened to 48px horizontally,
which costs no vertical space since they sit alone on their line.

## [2026-07-24] – Panel & Guidance Refinements (second device review)

Ten targeted notes from Aaron's review of the panels and guidance UI.

### Changed
1. **Landing:** the sync timestamp sits directly under RANDOM SCENE; the
   thought bubble is now 70% opacity so it reads as a quiet aside.
2. **Outline** uses full labels again — "Act 1", "Chapter 0" — on both
   desktop and mobile (only the per-scene status still collapses to an icon
   on mobile).
3. **Entry counts** in the Outline and Library headers are now a bold number
   in a gold circle, no text (both wired to the live count).
4. **Outline panel:** the sync button reads "Canon Sync" and matches the
   Changelog button's height; the Changelog icon is a sheet-of-paper glyph
   (`fa-file-lines`) so it no longer resembles the sync icon; "STORY OUTLINE"
   is right-aligned to the panel edge.
5. **Library panel:** category titles are ALL CAPS and clamp to 2 lines; the
   search field, the "Erica's Entries" banner, and the category review icons
   all share one height (44px), verified equal.
6. **Review markers:** the circle stays at the START of a category title but
   now sits at the END of an individual entry title. Tapping an entry's
   marker shows the reason; the next tap elsewhere clears that marker for
   good (persisted locally, and re-surfaced only if Sammy sends a genuinely
   new reason). Full reveal→clear→persist→re-surface cycle tested.
7. **Guidance rail:** all six tabs are now one icon-only strip on the right
   edge, touching each other, each only as wide as its icon (36px) but 44px
   tall for the tap. The editor sits to its left with more width.
8. **Guidance drawers** are more minimal: the duplicate outer title row is
   gone, the close (X) moved inline with the card's own icon+title row, a
   single separator line sits under that row, and the only other line is the
   2px window border.

### Verification notes
Re-audited at 440×956 across scene/drawer/notepad/stats/text-size/outline/
library/landing: 0 text under 12px, 0 measured WCAG AA contrast failures, no
horizontal scroll, no console errors. Library heights measured equal (44px);
category titles confirmed 1 line for short labels, 2-line clamp for long
ones. Desktop re-checked and behaviorally unchanged: full "Act 1" labels,
category titles NOT uppercased (mobile-only), review-marker click inert on
desktop (hover tooltip preserved), rail/nav hidden, 126px margins, 14px
editor, scene opens. The count-circle, "Canon Sync"/file-icon, and
end-positioned entry marker are intentional cross-platform changes Aaron
asked for, not regressions.

**Interpretation flagged:** item 5 says "review icons the same height as the
banner." Applied to the CATEGORY markers (which sit in the banner) = 44px;
entry markers were kept a compact inline 28px so a flagged entry row isn't
dominated by a 44px circle. Easy to change if all were meant.

## [2026-07-24] – Landing Nav Redesign (shared desktop + mobile)

Reworks the landing navigation into one deliberate cluster, used identically
on desktop and mobile (the old split — a desktop radial plus a separate
mobile reflow with `display:contents` + `order` — is gone).

### Changed
- **Tiff on top, buttons grouped beneath her**, all labels ALL CAPS.
- **WHERE I LEFT OFF** stays the hero: full cluster width, ~38% taller than
  the other buttons.
- **LAST COMPLETED** (left arrow at the left end) and **NEXT UNFINISHED**
  (right arrow at the right end) sit on one row directly beneath it, pinned
  to the outer edges.
- **RANDOM SCENE** centered below, matched to the half-button size.
- **SYNC is now a circle** floating in the empty square between the two half
  buttons — the circular-arrow icon fills the disc at low opacity with the
  SYNC label over it. It kept its `#landing-sync-btn` id and
  `fetchLatestFromCanon()` handler, so viewer-role gating and every call
  site still work untouched; the sync timestamp moved just below the cluster.
- Removed the obsolete mobile landing reflow CSS and the `#landing-sync-row`
  wrapper.

### Verification notes
Measured at 1440×900 and 440×956: GIF above the cluster, WHERE I LEFT OFF
352px wide / 66px tall, the two halves flanking a perfectly round 56px
circle centered on the cluster axis, RANDOM SCENE matched to the halves,
scene still opens from WHERE I LEFT OFF, editor still 14px on desktop, 126px
desktop margins intact, no horizontal scroll, no console errors, 0 sub-12px
text and 0 contrast failures in the landing.

**Open tradeoff, flagged not settled:** the half buttons come out ~39% of
WHERE I LEFT OFF, not the requested 50%. Exact-half + a visible central gap
+ a real circle can't all three hold in one row (two 50% halves leave zero
gap, so the circle would have to overlap them rather than sit in an empty
square). Kept the clean empty circle; the halves can be pushed wider if the
circle is allowed to shrink or overlap — Aaron's call.

## [2026-07-24] – Mobile Revision After First Device Review

Aaron reviewed the mobile build on the phone ("everything appears to function
fine, but we need to make adjustments") and gave ten notes. All ten are in.

### Changed
- **Two control tiers, both in `rem`.** `--tap` (2.75rem) for things touched
  while writing — bottom nav, editor toolbar, scene actions, landing nav.
  `--tap-sm` (1.875rem) for set-once chrome — header, panel headers, close
  buttons, sync, review markers. Because both are `rem`, lowering the UI text
  scale now genuinely **shrinks the buttons**, not just their labels; the
  scale range gained a downward half (**75–175%**, was 100–175%).
- **Header roughly halved**: logo mark 32px → 16px, header row 56px → ~34px.
- **Text buttons became icons on mobile** (Save Draft, Draft Log, Sammy,
  Proofreader, New, Save, Rename, Sync). Desktop keeps every word — the
  labels are wrapped in `.btn-text`, hidden only under `body.mobile-layout` —
  and each button carries an `aria-label` so nothing is lost to screen
  readers.
- **Nickname editing removed** entirely (button, handler, and the now-dead
  reference in `refreshAccountButton` that would have thrown without it).
- **The sync-first gate is gone.** `#landing-empty` was deleted; the landing
  is now the only front door, with Sync and its own timestamp moved to the
  bottom of it. "Stats & Progress" dropped from the landing nav, since Stats
  already sits in the header.
- **Stat teaser is now a thought bubble above Tiff's head** — CSS, not art,
  so the text reflows at any length and any UI scale rather than clipping
  inside fixed-size bitmap.
- **Bottom nav**: Focus moved to the left corner, ~36% larger, with the bar
  bulging up around it. Whichever view is open stays raised and gold until
  it closes — driven by what is actually open, so closing a sheet from its
  own X also drops the highlight.
- **Nothing covers the whole screen any more.** Sheets are bottom-anchored
  at 66dvh, leaving ~263px of the scene visible above them ("makes me feel
  like I've forgotten what I was working on"). Outline and Library are now
  **50% width** with a tap-to-dismiss scrim over the exposed half.
- **Outline condensed** for the narrower panel: "Act 1" → "A1", "Chapter 2"
  → "Ch2", and status words → distinct icons (check / exclamation / pen —
  still a shape per state, never colour alone), with padding stripped
  throughout. **Library** trimmed the same way, separators kept, and the
  review marker is now a smaller `fa-circle-exclamation` icon rather than a
  text "!".
- **Six guidance tabs split into two rails of three**, one per side, with
  the editor between them. Drawers open beside whichever rail they belong to.
- **Notepad and Draft Pad list headers** ("Your Notes", "Drafts") hidden on
  mobile, along with the outline hint text and both panel sync timestamps
  (the landing carries the timestamp now).

### Verification notes
Re-audited at 440×956 after every change: **0** text under 12px, **0**
measured WCAG AA contrast failures, **0** primary controls under 44px, **0**
sibling overlaps, no horizontal scroll, no console errors. Sheets confirmed
at 631px tall with 263px of scene still visible; both panels measured at
exactly 220px (50%); rails confirmed 3-and-3 with the editor between them.
Desktop re-measured and unchanged: 56px header, 32px logo, 126px margins,
side-by-side guidance columns, 320px panels, floating 480×380 Notepad, full
"Act 1"/"Review" wording, 14px editor, original red review pill.

**Still not verified:** nothing has run on real iOS Safari. The `:has()`
selector driving the panel scrim needs Safari 15.4+ (fine on her iOS 18
device, but unconfirmed on hardware), and Add to Home Screen remains
untested on a real phone.

## [2026-07-24] – Mobile Support: Touch Layout, Guidance Rail, Accessibility Pass

The tool's first real mobile experience, built for Erica's iPhone 16 Pro Max
(iOS Safari). This is an **interaction-model change for touch devices, not a
responsive CSS pass** — the desktop layout at ≥1280px with a mouse is
untouched. Everything new is additive and scoped behind a `body.mobile-layout`
class, so desktop cannot inherit any of it.

### Added
- **Capability-based mobile detection** — `isMobileMode()` keys off
  `(hover: none) and (pointer: coarse)`, deliberately **never width**. Her
  phone is 956px wide in landscape, which a conventional 768px breakpoint
  would have flipped into the desktop layout (draggable windows and all)
  just by rotating. A `matchMedia` change listener re-applies or fully
  clears the layout, so no half-applied state survives a mode change.
- **Right-wall guidance rail** (Aaron's drawer concept, 2026-07-24) — six
  vertical tabs in a 3×2 grid pinned to the right edge whenever a scene is
  open, staying put while the column scrolls. Tapping a tab pops that
  guidance card open as a drawer beside the rail; tapping again closes it.
  Replaces the desktop float/drag model without losing any content: the
  drawers **move** the real card node (as `floatGuidanceCard` always did),
  so there is still exactly one copy of each card's data.
- **Bottom navigation** — Home · Outline · Library · Notepad · Focus, all
  ≥44px and safe-area padded. **Focus** is the mobile reading of the desktop
  focus toggle: one tap closes every open drawer, panel and sheet. The
  header slims to logo + account + progress/stats in exchange.
- **All six former floating windows open as full-screen sheets**
  (Notepad, Draft Pad, Changelog, Proofreader, Send to Sammy, Stats),
  pinned between the header and the bottom nav, strictly one at a time —
  opening any sheet or panel closes the others via
  `mobileCloseEverything()`.
- **Vertical landing nav** — the radial arrangement re-flows into one
  centred column (GIF, then Where I Left Off, Next Unfinished, Last
  Completed, Random Unfinished, Stats), each 320×48 with 14px spacing.
  Nothing sits side-by-side at any phone size, including landscape.
- **Tap-to-reveal review reasons** — Sammy's `needsReviewReason` was
  reachable only by hovering a `title=` tooltip, which does not exist on
  touch, so on mobile that editorial information was completely
  unreachable. The `!` markers (both per-entry and per-category) are now
  44×44 bordered tap targets that toggle the reason inline. Tapping the
  category marker no longer also collapses the category.
- **`setMobilePreview(true/false)`** — a console/`?forceMobile=1` hook for
  exercising the mobile layout in a desktop browser, since the mobile media
  query cannot otherwise be triggered there.
- **Global UI text size** (`Text` in the bottom nav) — a slider plus large
  −/+ buttons, 100–175% in 5% steps, with a live preview and a reset. This
  is deliberately **separate from the editor-only zoom control**: it scales
  the root font size, so labels, buttons, panels and sheets all grow
  together rather than only the draft text. Stored per device (a phone and
  a laptop want different answers) and never synced. Applied as a
  percentage rather than a px value, so it multiplies the browser's own
  default text size instead of overriding it.
- **Add to Home Screen** — `apple-mobile-web-app-capable`, an app title,
  `theme-color`, a generated `assets/app-icon-{180,192,512}.png` (gold
  `#D4AF37` ground with dark `#3D2B1F` "BT", existing palette only), and a
  `manifest.webmanifest` so Android Chrome behaves too. Status bar style is
  `black`, **not** `black-translucent`, because translucent would put
  content under the status bar and the viewport meta deliberately does not
  use `viewport-fit=cover`. Saved to the home screen the tool launches
  standalone: no Safari chrome, and ~100px more vertical room.
- **Tap-to-expand for long scene text** — the scene summary and review note
  clamp to 3 and 2 lines with a "Show more"/"Show less" control (which
  hides itself when the text is short enough not to need it). Without this
  the editor started 948px down — past a full screen, meaning a scroll
  before every spontaneous sentence. Now 752px, with the formatting toolbar
  at 581px. No content is removed; the full text is one tap away.

### Changed
- **Aggressive autosave on mobile** — also fires on editor `blur`,
  `visibilitychange` (only when actually hidden), and `pagehide`.
  Backgrounding Safari mid-sentence is normal on a phone, and iOS suspends
  background tabs aggressively. Desktop autosave triggers are unchanged.
- **Drag/resize wiring is now lazy and desktop-only.** The twelve
  `make*Draggable`/`make*Resizable` calls moved out of `init()` into
  `ensureDesktopWindowWiring()`, which returns immediately in mobile mode —
  so a touch device never attaches a single mouse handler (iOS synthesises
  mouse events from taps, so "attached but unused" would not have been
  inert). Verified: on a mobile cold boot the wiring flag stays false.
- **Window geometry save/restore is skipped in mobile mode** — all six
  `save*State`/`restore*State` pairs bail out early. A phone session
  therefore cannot overwrite the desktop window arrangement, and a saved
  desktop rect cannot break a full-screen sheet. The sheets override the
  inline `left/top/width/height` with `!important` rather than clearing it,
  so desktop geometry is preserved rather than lost.
- **`adjustEditorZoom()` is mode-aware** — mobile runs 100–200% off a
  17px base (the 70% floor is useless for low vision, and the mobile
  default is readable without zooming at all). Desktop keeps 70–160% off
  the original 14px base.
- **Deferred to desktop on mobile**, per the "curation is a desktop task"
  decision: library entry deletion, the Notepad's Add-to-Library button,
  and the Changelog's All/Scene/Library/Structure filter tabs (mobile shows
  one chronological feed). Notepad note creation and editing stay fully
  available.
- **Accessibility scale-up (mobile only).** Nothing below 12px anywhere;
  editor 17px, body 16px, labels ≥14px; every touch target ≥44×44 with
  ≥8px spacing; all opacity-dimmed gold/cream labels forced to full-opacity
  `#D4AF37`/`#F4EDE4`. Inputs are ≥16px so iOS does not auto-zoom on focus.
  Hover-only tooltips are removed in favour of visible `.m-label` text.
  The viewport meta is untouched — pinch-to-zoom still works.
- **Review pill contrast** — the existing dark red on gold measures
  **4.34:1**, just under AA's 4.5:1. On mobile only, the pill text uses
  `#3D2B1F`, an existing palette pairing (it is what the gold primary
  buttons already use) measuring **6.39:1**. Desktop keeps the red. The
  gold background plus the literal word "Review" still carry the state, so
  no meaning rides on the text colour alone.
- **The mobile header now wraps at any width, not just narrow ones.** Found
  by screenshot, not by measurement: raising the UI text scale made the
  header's contents outgrow one row on a full-width phone, and because it
  was a fixed-height non-wrapping flex row they **collided into each other**
  rather than overflowing — so a right-edge overflow test saw nothing wrong.
  An overlap check (do sibling boxes actually intersect?) was added to the
  verification pass alongside the overflow check.
- The landing GIFs now carry `loading="lazy"` so a phone on cellular does
  not eagerly pull art it will not show. **The existing random pick of one
  of the three per landing render is unchanged and confirmed working** —
  30 consecutive renders drew all three — and only the chosen GIF is ever
  fetched, so a landing costs ~1.3MB, not 3.5MB.

### Verification notes
Tested in a real browser (Chromium, live-loaded Tailwind + Font Awesome,
real canon fetched from GitHub — 4 scenes) at **440×956**, **956×440**,
**375×812**, **220×478** (the geometric equivalent of Safari Page Zoom at
200% on a 440px phone), and **1440×900** desktop. Measured, not eyeballed:
- **Zero** text below 12px, **zero** WCAG AA contrast failures (computed
  ratios with alpha compositing against the real effective background), and
  **zero** touch targets under 44×44 — across all ten views (scene editor,
  all six sheets, outline panel, library panel with categories expanded,
  landing) at every viewport above.
- **No horizontal page scroll** at any of the five sizes. Three real
  overflow bugs were found and fixed this way: the header's right group
  clipped the STATS button (body has `overflow:hidden`, so clipped meant
  unreachable), the editor toolbar's inner groups did not wrap at high
  zoom, and the nav's five flex buttons squeezed to 43.66px at 200% zoom.
- **Landscape at 956px stays in mobile layout** — confirmed
  `isMobileMode()` true, margins 0, guidance columns collapsed, nothing
  side-by-side. Rotated both directions with a drawer open; no stale or
  half-applied geometry, drawer stayed inside the viewport.
- **Desktop geometry is safe**: seeded six known window rects, ran a full
  mobile session opening and closing all six sheets, and confirmed every
  stored rect was byte-identical afterward and that sheets ignored them.
- **Autosave**: verified saves on blur, on `pagehide`, on
  `visibilitychange` only when hidden (not when visible), and that manual
  Save Draft still writes.
- **Viewer role**: on mobile a viewer gets `role-viewer`, non-editable
  editor and notepad, hidden sync button, zero visible `.primary-only`
  controls, and **no server write is attempted** (confirmed by intercepting
  `fetch`).
- **Desktop regression**: measured 126px margins, side-by-side guidance
  columns, 14px editor, 10.4px status badges, original red review pill,
  visible tooltips, sticky summary/status bars, 48px status bar, 70–160%
  zoom range, and working drag **and** resize (dispatched real mouse
  sequences; geometry moved and persisted). Confirmed by direct
  before/after comparison, not assumption.
- Zero console errors throughout (only Tailwind's standard CDN advisory).
- Diff review: the only reference to protected sync/encryption/auth code in
  the whole change is a **call site** — `pushDraftToServer(...)` on the new
  blur/visibilitychange trigger. No protected function definition was
  modified.

- **UI text scale** verified at 100/125/150/175%: root font tracks
  (16/20/24/28px), the header re-measures and sheets follow it exactly
  (94→102→114→130px), and at the 175% maximum there are still zero
  sub-12px, zero contrast failures, zero undersized targets and no page
  overflow. Cleared entirely on desktop (no inline root font-size).

**Not verified — stated plainly:**
- **Add to Home Screen was not tested on a real device.** The meta tags,
  manifest and icons are in place and the icon renders correctly as a file,
  but actually saving the tile on an iPhone, its icon appearance on the
  home screen, and standalone launch behaviour (including how the header
  sits under the status bar) are all unconfirmed. This is the first thing
  to check on the phone.
- **Nothing was tested on real iOS Safari or a real iPhone.** All testing
  was desktop Chromium. Specifically unverified: `contenteditable` caret
  visibility and scroll-into-view with the on-screen keyboard up
  (historically buggy on iOS), the `visualViewport`-driven keyboard padding,
  `env(safe-area-inset-*)` behaviour around the Dynamic Island and home
  indicator, real Safari Page Zoom (only its geometric equivalent), actual
  pinch-to-zoom gestures, and real touch-vs-mouse event behaviour.
- **The `(hover: none) and (pointer: coarse)` query was never observed
  matching**, because this environment is mouse-only. The query string was
  confirmed to parse validly (`matchMedia().media` echoes it back rather
  than `not all`), and all mobile behaviour was exercised through
  `setMobilePreview()`, which drives the identical code path. But the
  detection itself firing on her actual phone is unconfirmed.
- Real cross-device sync round-trips were not re-tested; the sync layer was
  not modified, only invoked on additional triggers.
- Landscape shows the six-tab rail clamped to ~310px tall with internal
  scrolling — reachable, but it does scroll. Not tested by a human hand.

Wires the Phase 1 UI to real, shared, cross-device persistence — **with no
backend**, using the public `big-tiff-data` repo as a JSON store (static
GitHub Pages hosting; the old Cloudflare assumption was wrong). Login becomes a
real (soft, client-side) hash check; drafts sync to the cloud.

### Added
- **`big-tiff-data` repo (public), seeded** with `accounts.json`,
  `drafts.json`, `checkpoints.json`, `stats.json` scaffolds + a README
  explaining the privacy model.
- **`account-hash-generator.html`** — a standalone local page that produces
  PBKDF2-SHA256 salted hashes and a ready-to-commit `accounts.json`, so real
  passwords never leave the setter's browser.
- **`SETUP.md`** — the three one-time setup steps (create accounts, turn on
  Erica's write token, promote the tool to the launch site's `/app`).
- **Data layer in `writing.html`:**
  - AES-GCM `encryptDraft`/`decryptDraft` with an embedded key (drafts stored
    as ciphertext in the public repo = "lightly private").
  - `verifyPassword` (PBKDF2-SHA256, 100k iters) — real client-side login vs
    `accounts.json`, replacing the Phase 1 mock. `submitLogin` is now async.
  - `syncFromDataRepo()` runs on every load (via `bootstrapData()`): public
    raw fetch of drafts/checkpoints/stats so every profile opens to the same
    shared state — no manual Sync needed. `validateSessionAgainstAccounts()`
    drops stale sessions and treats `accounts.json` as authoritative for role.
  - Write layer (`pushToDataRepo`, `getFileSha`) via the GitHub Contents API
    using Erica's pasted token; `pushDraftToServer` (debounced from autosave,
    immediate from Save) re-reads + merges before writing so other scenes are
    never clobbered, and the API `sha` gives free conflict detection.
  - `pullDraftFromServer()` in `loadCurrentDraftForScene`: adopts the remote
    copy only when local is empty; otherwise, on a real divergence, raises the
    **conflict banner** — never a silent overwrite. Resolving it saves the
    not-kept version as a Draft Pad checkpoint first (nothing is ever lost).
  - `syncStatsToRepo()` pushes scenes-completed / word-count / streak / an
    open-ended event log to `stats.json`; the progress pill, stats window, and
    teaser now read real shared stats.
  - **Write-token first-run walkthrough** (`token-setup-modal` +
    `saveWriteToken()` which verifies the token with a real authenticated read
    before storing it, browser-only). Re-openable from the account menu.
- **Launch page (`big_tiff_launchpage`):** the "follow the light" dialog's stub
  submit handler is replaced with a real `accounts.json` hash check that writes
  the same `bigtiff-session` localStorage key the tool reads, then redirects to
  `/app`; an in-dialog `.portal-error` shows failures. `writing.html` is
  promoted to `app/index.html` (same origin as the login → shared session).

### Verification notes
- Verified in a real browser against the **live seeded repo**: AES round-trip
  (incl. unicode/emoji), `verifyPassword` correct/incorrect, real fetch of all
  four JSON files, real login (correct → primary session + token prompt; wrong
  → error). **Full remote round-trip:** a browser-encrypted draft committed to
  `big-tiff-data`, re-fetched, and decrypted back to the exact original;
  `pullDraftFromServer` adopt-vs-conflict paths; conflict resolution saving the
  loser as a checkpoint (nothing lost). Launch-page `authenticate()` verified
  live (parity + a stubbed-accounts success writing the exact session shape).
  `api.github.com` CORS reads confirmed from a `file://` origin.
- **Caveat:** the browser's actual **write PUT** to GitHub (`pushToDataRepo`)
  could not be exercised end-to-end here because injecting a real token into the
  sandbox browser was blocked — it's verified **by parity** (identical endpoint/
  body to a successful git-based commit of the same encrypted payload, plus the
  confirmed CORS read). True write E2E happens the first time Erica pastes her
  token via the setup walkthrough. Also: cross-page session sharing relies on
  same-origin `/app` hosting, which only exists once the launch-page branch is
  deployed. Accounts are empty until the user runs the hash generator.

## [2026-07-23] – Landing/Home Screen, Accounts UI, Stats & Role Gating (Phase 1 of 2)

Phase 1 of the larger "accounts + live persistence" effort (full plan lives
outside the repo in the session plan file). This phase is **all client-side
and browser-testable now** — it adds the UI/UX and local logic, with mocked
session/stats standing in for the real GitHub-backed backend that Phase 2 will
wire up. Nothing here talks to a server yet.

**Important architecture note recorded this session:** the live site
(`big_tiff_launchpage`, bigtiffsworld.com) is hosted on **GitHub Pages
(static)** — its `worker.js` is unused Cloudflare leftover scaffolding. There
is no backend. Phase 2's persistence will therefore be GitHub-native: a public
`big-tiff-data` repo holding JSON (drafts encrypted with an embedded AES key =
"lightly private"; reads are free public fetches; only Erica's browser writes,
via a GitHub token she pastes once). Login is a soft client-side gate, not real
security. This replaces the earlier (wrong) Cloudflare/D1 assumption.

### Added
- **Landing / home screen** (replaces the old `#no-scene-placeholder` text).
  Shown when no scene is selected. Two states: an "empty" one-tap **Sync
  Canon** call when no canon is loaded yet, and — once canon is loaded — a
  radial nav: **Where I Left Off** (center/main), **Last Completed** (left),
  **Next Unfinished** (right), **Random Unfinished** (top), **Stats &
  Progress** (bottom). A random Tiff GIF (`assets/TiffGif_Doodling|Laying|
  Sitting.gif`, new `assets/` folder) is picked per render. New `goHome()`
  clears the scene, closes all windows, and returns here. `selectSceneFromCanon`
  now records `last-selected-scene-id` so "Where I Left Off" works.
- **Rotating stat teaser** at the top of the landing screen (random snippet
  per render; mocked stats in Phase 1).
- **Header account button (dual-state)**, replacing the dead `showLoginModal()`
  stub: logged-out opens an in-tool login modal; logged-in shows the user's
  **nickname** (defaults to username, editable display-only via a pencil — the
  username never changes) plus a logout menu. A **rotating salutation**
  ("Greetings, {nickname}!", "Shine on, {nickname}!", …) shows near the logo.
- **Stats & Progress window** (`showStoryStats()`, replacing the dead stub) —
  a draggable/resizable floating window mirroring the Draft Pad pattern, with
  counters (scenes done / words / streak) and an achievement grid. Achievement
  *content* is intentionally a future curated task; the grid renders whatever
  the stats object provides. The header **progress pill** (previously the
  hardcoded "Act 1 • 42%") is now wired to real local data via
  `refreshProgressPill()` (overall % of scenes marked complete).
- **Viewer-role UI gating** (`applyRoleGating()`): a `role-viewer` body class
  hides `.primary-only` controls (Add-to-Library, delete-note, library-entry
  delete), makes the scene editor read-only, and hides Sync Canon — with an
  **AaronF exception** that re-enables Sync for that one viewer account.
  Notepad create/edit stays available to viewers. A TEMP `?mockRole=` /
  `?mockConflict=` URL flag drives Phase-1 testing (removed in Phase 2, when a
  real session replaces it). Code comments note this gating is UX-only; the
  real enforcement in Phase 2 is that viewers hold no write token.
- **Draft-conflict banner** (UI only this phase): a non-blocking compare/choose
  banner ("keep this device's" vs "use the other device's" version) modeled on
  the Draft Pad's confirm-before-overwrite pattern, to be wired to real
  server-conflict detection in Phase 2. It never silently overwrites; the
  losing version is to be auto-saved as a checkpoint.

### Verification notes
- Verified in a real browser (this session's sandbox **did** load the Tailwind/
  Font Awesome CDNs, unlike the prior session) via direct DOM + computed-style
  assertions and a real canon fetch (4 scenes pulled from GitHub): logged-out/
  no-canon defaults; canon sync flips the landing from empty→radial with a
  randomized GIF (confirmed the GIF loads, 1800×1098); all five nav buttons
  (including the "no completed scenes yet" and review-fallback paths);
  `last-selected-scene-id` persistence + Where-I-Left-Off; login as primary vs
  viewer vs AaronF; nickname edit + greeting; Stats window open/render/close;
  conflict banner show/resolve; logout; and a regression pass (autosave still
  writes `current-draft:<id>`, Notepad/Draft Pad/closeAll still work, no console
  errors). A real visual screenshot confirmed the radial layout + animated GIF.
- **Caveats (per project convention):** (1) the preview pane locks the viewport
  to ~280px wide and strips URL query strings, so the desktop-width visual
  required temporarily neutralizing the tool's pre-existing fixed 252px center
  margin (a runtime-only override, not a code change), and the `?mockRole=` URL
  entry was verified by injecting the query via `history.replaceState` rather
  than a real navigation — the underlying param logic and gating were confirmed
  either way. (2) Everything server-related (real login, cross-device draft
  sync, real stats/achievements) is **mocked** and unverified end-to-end until
  Phase 2 provisions the `big-tiff-data` repo and wires real GitHub I/O.

## [2026-07-23] – Focus Mode Split into Writing Focus / Reading Focus

### Added
- **Focus button replaced with a two-step segmented toggle** (Priority 1
  item from the handoff doc). Two icons - a pen (`fa-pen-to-square`,
  closest available Font Awesome Free glyph; there's no literal
  "hand holding a pen" icon in the free set) for Writing Focus, and an open
  book (`fa-book-open`) for Reading Focus - flank a small pill track/knob.
  Clicking an icon only **arms** that mode (moves the knob, highlights the
  icon); clicking the track itself **activates** whichever mode is armed.
  Selection persists after activation. Matches the confirmed spec: select,
  then a separate deliberate click to fire, rather than one click doing both.
- **`writingFocusMode()`** - clears every floating window/panel (Notepad,
  Draft Pad, Changelog, Proofreader, Send to Sammy, left/right side panels)
  and returns all guidance ("story point") cards to their home positions,
  fully open - extends the old `focusMode()`, which only closed Notepad +
  side panels + floating guidance cards and left Draft Pad/Changelog/
  Proofreader/Send to Sammy untouched if they were open.
- **`readingFocusMode()`** (new) - same clear-everything base as Writing
  Focus, plus collapses all 6 home guidance cards to their header-only row
  (reusing the existing `toggleGuidanceCardMinimize()`), to reduce on-screen
  text while reading a full draft.
- New shared `closeAllFloatingWindows()` helper factors out the "hide
  everything" logic so both modes share one code path.

### Fixed
- **Guidance card floating windows (`.guidance-float-window`) were visibly
  rendered and click-through-able on every page load**, despite having
  `class="hidden"` - found while verifying Focus mode in a real browser
  (fresh, untouched page load; confirmed via computed `display: flex` on
  all 6 float windows despite the `hidden` class being present). Same root
  cause as the `.notepad.hidden` bug fixed 2026-06-22: the class sets its
  own `display: flex`, which can out-cascade Tailwind's `.hidden` utility
  depending on stylesheet parse order. Fixed with the same
  `.guidance-float-window.hidden { display: none !important; }` pattern.
  This was a real, pre-existing bug independent of the Focus mode work,
  just never caught because nobody had screenshotted a truly fresh load.

### Verification notes
- Verified in a real headless-Chromium browser (Playwright): opened every
  closable window + floated a guidance card, confirmed Writing Focus closes
  all of them and Reading Focus additionally collapses all 6 home cards;
  confirmed re-activating Writing Focus afterward un-collapses them (so
  "home positions, open" holds even after a prior Reading Focus click);
  confirmed selecting an icon alone never fires anything, only clicking the
  track does. Verified via direct DOM state (classes + computed `display`),
  which doesn't depend on Tailwind.
- **Caveat, stated explicitly per project convention:** this session's
  sandbox blocks outbound access to `cdn.tailwindcss.com` and
  `cdnjs.cloudflare.com` at the network policy level (confirmed via proxy
  status logs, not just a timeout), so Tailwind's CDN script never loaded
  in any test run here - screenshots taken in this sandbox show an unstyled/
  mis-flowed page and were **not** usable for visual/layout confirmation of
  the new toggle's appearance. The DOM-state checks above are unaffected by
  this (plain CSS classes/computed styles, no Tailwind dependency), but a
  real visual check of the toggle's look in a normal browser with internet
  access is still outstanding and should happen before calling this fully
  done in the strict "tested in a real browser" sense this project uses.

---

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
