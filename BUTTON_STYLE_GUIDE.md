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

Three **named, deliberate exceptions** sit outside this floor on purpose —
don't "fix" them without asking Aaron first:
- The guidance-rail tabs (`.rail-tab`) and the scene-status circle
  (`.status-dropdown-trigger` in mobile) are **3.5rem/56px**, one size up
  from `--tap`, so the two edge-mounted control banks read as the same
  visual weight as each other. This is a deliberate exception to `--tap`,
  not a bug.
- The rail's collapse control (`.rail-collapse-toggle`) is **60×140px**
  as of 2026-08-02 — it left the `--tap-sm` tier entirely when Aaron
  respecified it as a **folder tab** centered on the rail's long left wall
  rather than a small arrow at the strip's foot. 140px = 2.5 rail icons,
  60px = twice the old toggle's 30px depth. It is not a "set once and
  ignore" control any more; it's the handle you grab to reopen a collapsed
  rail, so the generous size is the point. Don't shrink it back toward
  `--tap-sm` without asking.

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
| `.text-tool-btn` (editor toolbar: Proofreader, B/I/U) | `--tap` (44px) | `1rem` | mobile |
| `.status-actions-bank > button` (Save/Draft Log/Sammy) | `--tap` (44px) | `1.21875rem` (2026-08-02; was `0.9375rem` inherited from `.control-bar-item`) | mobile |
| `.rail-tab` (guidance drawer triggers) | 3.5rem (56px, exception above) | `1.3rem` (2026-08-02; was `1rem`) | mobile |
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
