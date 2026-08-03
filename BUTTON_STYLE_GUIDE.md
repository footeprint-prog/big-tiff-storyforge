# Button & Accessibility Style Guide — `writing.html`

Single reference for button sizing, icon sizing, and accessibility minimums
across **both** layouts the tool renders — desktop (mouse, `body` without
`.mobile-layout`) and mobile (touch, `body.mobile-layout`). The two layouts
diverge a lot visually; this doc is what keeps new buttons in either one
consistent with what's already shipped, rather than each round inventing a
new size by eye. Added 2026-07-27 after the guidance-rail icons drifted to
1.9rem (nearly double the editor toolbar) with no doc catching it.

Read `AGENTS.md` and `Mobile_Port_Handoff.md` first for project-wide rules
this doc doesn't repeat (e.g. never gate mobile detection on width).

---

## 1. The two tiers (mobile)

Defined once as CSS custom properties, scaling together if the user's UI
text-scale setting changes:

| Var | Size | Meaning | Examples |
|---|---|---|---|
| `--tap` | 2.75rem / 44px | Anything touched **while actively writing** | Editor toolbar (`.text-tool-btn`), scene-status action bank (Save/Draft Log/Sammy), bottom nav |
| `--tap-sm` | 1.875rem / 30px | Chrome she **sets once and ignores** | Panel close buttons, drawer-close (`.m-drawer-close`), copy-to-clipboard (`.guidance-copy-btn`), status-actions arrow tab |

Rule of thumb before adding a new mobile button: is this something Erica
taps repeatedly mid-writing-session, or a set-once toggle? That answers
which var to use — don't invent a third size.

Five **named, deliberate exceptions** sit outside this floor on purpose —
don't "fix" them without asking Aaron first:
- The guidance-rail tabs (`.rail-tab`) and the scene-status circle
  (`.status-dropdown-trigger` in mobile) are one size up from `--tap`, so
  the two edge-mounted control banks read as the same visual weight as each
  other. The status circle is `3.5rem/56px` square; the rail tabs were too
  until 2026-08-02, when they became **`2.95rem/47.2px` wide × `3.5rem/56px`
  tall** (no longer square — see the update below). This is a deliberate
  exception to `--tap`, not a bug.
  **2026-08-02 update:** Aaron asked for the icon's top/bottom/left margins
  to match (right stayed at the 8.8px set the same day, above — not part of
  this ask). With height fixed at 56px, the common-case 20.8px icon already
  centers to a ~17.6px top/bottom margin; matching left to that only
  required a 47.2px-wide box (17.6 + 20.8 + 8.8), instead of the old 56px
  square with ~26px of dead space on the icon's editor-facing side. Removing
  that dead space is also what makes the rail narrower — same change, not a
  separate one. Width now lives in `--rail-tab-w` (a shared custom property,
  also read by the guidance-drawer offset and the JS drag clamp) rather than
  a repeated literal.
- The rail's collapse control (`.rail-collapse-toggle`) is **24×112px** as of
  2026-08-02 (height revised same day, was 96px) — it left the `--tap-sm`
  tier entirely when Aaron respecified it as a **handle** on the rail's long
  left wall rather than a small arrow at the strip's foot. It was briefly
  60×140 (2.5 rail icons tall) the same day; Aaron then pointed at **iOS's
  own hidden-PiP-window handle** — the slim translucent pill on a phone
  screen's right edge — and asked for the same size and shape (~24×96 CSS px,
  4:1, fully rounded outboard edge, hence the 12px radius = half the width).
  **Later the same day, height was revised again to 112px** (`calc(2 *
  var(--rail-tab-h))`, "as tall as 2 rail-drawer boxes") — this deliberately
  breaks the 4:1 iOS-ratio fidelity the previous revision matched; Aaron's
  explicit new number is what governs now, not the ratio. Width (24px) and
  the 12px radius are unchanged (the radius derives from width, not height).
  **This is the one control in the tool that sits below the 30×30 chrome
  touch floor in one dimension** (24px wide). It is a deliberate,
  Aaron-specified exception, made on the reasoning that it is edge-anchored
  and well over 100px tall, so the hard-to-miss axis is the long one. Don't
  "fix" the 24px on accessibility grounds without asking him — and don't
  cite it as precedent for shrinking anything else.
- The **editor-toolbar zoom scaler** (`#editor-zoom-control`, the −/100%/+
  pill) is a second, narrower deliberate exception, added 2026-08-02: its
  total height is capped at exactly `--tap`/44px to match its sibling
  toolbar buttons, which pushes its own −/+ buttons a few px under 44px
  when measured in isolation (their `min-height` is overridden to `auto`
  inside this control specifically, so they don't fight the pill's own
  fixed height). The pill as a whole still meets the 44px floor; only the
  two glyphs inside it individually read slightly under it.
- **`.review-flag-btn`** (the "!" review marker, Library panel only —
  category-header and per-entry contexts both) is **1.5rem/24px** as of
  2026-08-03, matching `.count-badge` (the gold entry-count circle) —
  Aaron: "same size as the entry-count circle," explicitly "ALL of" the
  marker's occurrences in that panel. Was three different sizes before this
  (30px generic fallback, 44px at category headers via `--lib-banner-h`,
  28px per-entry) — none of them matched `.count-badge` and none matched
  each other. Below the 30×30 chrome floor, same category of exception as
  the rail handle above. Icon `font-size` inside the circle is untouched in
  every context — this only changed the box.
- **`#outline-close-btn` / `#library-close-btn`** (the Outline/Library
  panels' own close arrows) had their icon `font-size` raised `1.125rem` →
  `1.5rem` the same day, also to match `.count-badge` — the
  `fa-arrow-alt-circle-left/right` glyphs are already circular outlines, so
  matching font-size to 1.5rem lands the rendered icon at very close to
  24px without adding the background/border `.count-badge` has. The
  button's own tap target is unaffected — the generic panel-button rule
  already floors it at `--tap-sm` independent of the icon's size, so this
  one isn't actually a touch-target exception, just an icon-size note
  worth keeping next to the others above.
- **The editor toolbar itself** (`#editor-toolbar .text-tool-btn` — B/I/U,
  Proofreader, Copy, Paste, Text Size — and `#editor-zoom-control`, the
  −/100%/+ pill) left the `--tap` PRIMARY tier entirely on 2026-08-04,
  shrinking 44px → 22px on an explicit Aaron ask ("reduce size of text
  editor tool buttons and scaler bar by 50%"). Unlike every other
  exception in this section, these buttons ARE touched constantly while
  writing — this is the one deliberate departure from "primary-tier
  buttons stay at 44px," confirmed directly via AskUserQuestion ("the
  whole button box," not just the glyph) before being made. **Revised
  2026-08-05:** Aaron asked to enlarge again, 22px → 36px. **Revised again
  2026-08-05 (later round):** 36px → **40px**, which is where it sits now
  — still below the 44px floor, but the closest it's been to it since
  leaving the tier. Don't restore either the 44px floor or re-shrink
  toward 22px without asking again; these sizes have already been
  explicitly chosen and revised multiple times.
  Copy/Paste were **removed entirely** from this toolbar the same later
  round (not just hidden — the buttons, their divider, and the now-dead
  `pasteToEditor()` function are gone from the DOM/code). Bold/Italic/
  Underline were also fused into a single bordered rectangle with 1px gold
  separators between them, replacing their previous individually-bordered,
  gapped look — mobile-only; desktop's copy of this toolbar is untouched.
- **Notepad and Draft Pad (mobile) reclassified 2026-08-05 (fourth round),
  as part of a full redesign** (see CHANGELOG for the "practically
  unusable" measurements that drove it). Every button in both panels had
  been blanket-forced to 44px/PRIMARY, regardless of how often it's
  actually tapped. Now: `#notepad-toolbar-format-group .text-tool-btn`
  (B/I/U) is **40px**, fused-rectangle style, matching the main editor
  toolbar's own treatment above; the zoom pills
  (`#notepad-zoom-control`/`#draftpad-zoom-control`) are also **40px** to
  match; everything else in either panel (New, Delete, Add-to-Library,
  Save, Rename, Load-into-editor, the back-to-list button) dropped to
  **`--tap-sm`/30px, CHROME tier** — all of them are "set it and move on"
  actions, not controls touched constantly while writing. Copy/Paste
  removed from both panels' mobile toolbars, same as the main editor.
  Desktop's side-by-side layout and button sizes are untouched.

## 2. Icon (glyph) size is separate from box size — don't conflate them

A button's touch target (the box) and its icon's `font-size` (the glyph)
are two different numbers. Enlarging the box for a touch target does
**not** mean the glyph should scale up proportionally — a big box with a
small, centered glyph reads calmer and matches the rest of the tool better
than a glyph that fills the box.

**Reference icon sizes — keep new icon buttons within this range unless
there's a specific reason to differ (state that reason in a comment):**

| Class | Box size | Icon `font-size` | Layout |
|---|---|---|---|
| `#editor-toolbar .text-tool-btn` (B/I/U, Proofreader, Text Size) | 40px (exception, see §1 — was `--tap`/44px, revised three times; Copy/Paste removed 2026-08-05 later round) | `1rem` | mobile |
| `.status-actions-bank > button` (Save/Draft Log/Sammy) | `--tap` (44px) | `1.21875rem` (2026-08-02; was `0.9375rem` inherited from `.control-bar-item`) | mobile |
| `.rail-tab` (guidance drawer triggers) | 2.95rem × 3.5rem (47.2×56px, exception above; was 56×56 square) | `1.3rem` (2026-08-02; was `1rem`) | mobile |
| `.m-drawer-close` | `--tap-sm` (30px) | `1.15rem` | mobile |
| `.guidance-copy-btn` | `--tap-sm` (30px) | `1rem` | mobile |
| `.control-bar-item` (desktop toolbar/status row) | 30px height | `12px` (`0.75rem`) | desktop |
| `.guidance-float-btn` (minimize / float-out / snap-back) | auto | `11px` | desktop |
| `.side-icon-btn` (Outline/Library nav) | 52px | inherits FontAwesome default | desktop |

**2026-07-27 correction:** `.rail-tab` icons were enlarged to `1.9rem` on
2026-07-26 to "squeeze dead space" — this overshot and made them read as a
different, heavier control family than the editor toolbar and status
buttons next to them. Reverted to `1rem` per Aaron. If dead space around an
icon is a real problem again, shrink the box's internal padding first,
not the glyph.

**2026-08-02 update — the icon-margin item above is now settled, by
Aaron's own number.** He asked for a flat **+30% glyph** on both the rail
tabs (`1rem` → `1.3rem`) and the action bank (`0.9375rem` →
`1.21875rem`), explicitly keeping every margin, padding, and border as-is.
This is not a re-run of the reverted 2026-07-26 attempt: that one was a
2.5× guess made in place of asking, this is the requested ratio, and it
lands well short of the size that read as a different control family.
Both boxes are untouched (56px rail tab, 44px `--tap` action button), so
touch targets and the arm's circle-driven column geometry are unchanged —
confirmed by measurement, not by eye.

Also 2026-08-02: **Draft Log renders the letters `DL` instead of its
`fa-layer-group` icon on mobile only.** Implemented the same way the
status circle does its short codes — both forms sit in the DOM
(`.btn-text` / new `.btn-code`), CSS picks one per layout, no JS
branching, and the button's `aria-label` still carries "Draft Log" so the
accessible name never changed. Desktop keeps icon + full text. If you add
another letter-code button, reuse `.btn-code` rather than inventing a
parallel class.

## 3. Accessibility minimums (both layouts)

Re-run this checklist after **any** visual change to a button — it already
caught real bugs during the mobile port (see `Mobile_Port_Handoff.md`'s
"Accessibility audit method" section for the reusable script shape):

- **Text ≥ 12px** computed font-size for anything readable, icons included
  where the icon carries meaning alone.
- **WCAG AA contrast** (4.5:1 normal text / 3:1 large text or icons),
  computed by compositing every translucent layer from the element up to
  its real background — not just checking the two nominal colors in
  isolation. Dimmed gold/cream (`text-[#D4AF37]/70` etc.) that passes on
  desktop's lighter chrome can fail on mobile's darker one; mobile forces
  full opacity on these (see the `[class*="text-[#D4AF37]/"]` override) —
  match that pattern rather than adding a new one-off color.
- **Touch targets:** ≥30×30 for chrome-tier (`--tap-sm`), ≥44×44 for
  primary-tier (`--tap`) interactive elements. Desktop has no touch-target
  floor, but don't shrink a control below what's comfortable to click
  either — 30px height (`.control-bar-item`) is the desktop floor in
  practice.
- **Never** let a status/meaning ride on color alone (see the Review pill:
  gold background + the literal word "Review", not just a color swap).
- **Verify with real measurement**, not eyeballing: `getBoundingClientRect()`
  for target size, computed-style contrast math for color — screenshots
  alone have already caused missed regressions on this project.

## 4. Before adding or resizing any button

1. Pick the tier (`--tap` vs `--tap-sm` on mobile; match an existing
   desktop class) based on *how Erica uses it*, not how it looks first.
2. Pick an icon `font-size` from the reference table in §2 — don't eyeball
   a new number.
3. Run the accessibility checklist in §3.
4. Verify desktop is unchanged (screenshot/measure both layouts) — this is
   not optional per `Mobile_Port_Handoff.md`.
5. Update this doc's table if you're introducing a genuinely new class or
   deliberately breaking the pattern (and say why, like the `.rail-tab`
   exception above) — a style guide that silently drifts from the code is
   worse than no style guide.
