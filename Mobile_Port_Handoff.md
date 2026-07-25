# Big Tiff StoryForge — Mobile Port Handoff

## For: whoever picks up mobile work next (agent or human)
## From: Claude session, 2026-07-24 – 2026-07-25, at Aaron's direction
## Branch: `claude/mobile-port` (7 commits ahead of `main`, clean working tree)
## Live preview: https://footeprint-prog.github.io/big-tiff-storyforge/writing.html

---

## Read this first, in this order

1. `AGENTS.md` — hard rules for this repo generally (don't re-derive).
2. The mobile-port task brief that started this branch — it's not a
   separate file, it was the original prompt; the north-star constraints
   from it (below) still fully apply to any further mobile work.
3. `CHANGELOG.md`, entries dated 2026-07-24/25 (there are seven, newest
   first — "Add mobile support" is the foundational one, the rest are
   device-review revision rounds). Each has a **Verification notes**
   section stating exactly what was tested and how, and most have a
   **flagged, not settled** or **intentional exception** callout — read
   those before assuming something is a bug.
4. `CHECKLIST.md` — the "Mobile version of the tool" item is now marked
   done, with a pointer back to this caveat.

Do not re-derive anything below; it cost real back-and-forth with Aaron to
establish.

---

## What this branch is

A full mobile/touch interaction layer added to `writing.html` (single file,
no build step, unchanged architecture) for Erica's iPhone 16 Pro Max. Not a
responsive CSS pass — a second interaction model living behind
`body.mobile-layout`, toggled by a capability query
(`(hover: none) and (pointer: coarse)`), **never by width** — her phone is
956px wide in landscape, which would fool a width breakpoint into desktop
layout.

Desktop (`>=1280px`, mouse) is verified unchanged after every single commit
in this branch. That check is not optional for future changes either.

## How to test it yourself (no real phone required for CSS/logic work)

There's a `setMobilePreview(true|false)` function exposed on `window`,
callable from the console or `?forceMobile=1` in the URL. It flips the same
`isMobileMode()` the real media query drives, so it exercises the identical
code path — this is how the entire branch was verified in a mouse-only
browser. Pattern used throughout: `setMobilePreview(true); await
fetchLatestFromCanon(); goToWhereILeftOff();` then measure real
`getBoundingClientRect()` / `getComputedStyle()`, never eyeball a
screenshot alone (though screenshots caught two real bugs numeric checks
missed — see CHANGELOG "Scene View Redesign" and "Landing Nav Redesign"
entries for what those were and why the audit script was blind to them).

## The one deployment fact you need

`https://footeprint-prog.github.io/big-tiff-storyforge/` is **GitHub
Pages turned on for this dev repo**, pointed at this branch — separate
from and not touching the real live site (`bigtiffsworld.com`, a different
repo, `big_tiff_launchpage`, still on `main`/desktop-only, confirmed
untouched). It rebuilds within ~1 minute of every push to this branch.
**Promotion to the real site is still a deliberate separate manual step**
per the original brief — do not do it without being asked.

---

## Architecture established this branch (don't rebuild, extend)

- **Detection:** `isMobileMode()` / `applyMobileLayoutClass()`. Toggling
  adds/removes `body.mobile-layout`; a `matchmedia` change listener plus
  `setMobilePreview()` both funnel through it.
- **Two-tier control sizing**, both in `rem` so a UI text-scale change
  actually resizes controls, not just labels:
  - `--tap` (2.75rem/44px) — anything touched while writing: bottom nav,
    editor toolbar, the three scene-action buttons, guidance rail, landing
    nav, proofreader go-to arrow.
  - `--tap-sm` (1.875rem/30px) — set-once chrome: panel headers, close
    buttons, changelog/sync icons.
  - Two **named, deliberate** exceptions below this floor exist (status
    pill's dropdown options, the show-more/less chevron height) — both
    requested explicitly across two review rounds. Don't "fix" them without
    asking; see the last two CHANGELOG entries for exact numbers.
- **Runtime-measured CSS variables**, not hardcoded, because header/nav
  height change with UI text scale and content:
  `--mobile-header-h`, `--mobile-nav-measured` (kept **separate** from
  `--mobile-nav-h`, which the nav's own buttons consume — merging these
  two created a real feedback-loop bug, fixed and documented in the "Scene
  View Redesign" entry; don't recombine them).
- **Global UI text scale**, `getUiScale()`/`setUiScale()`, 75–175%, stored
  in `localStorage['bigtiff-ui-scale']`, separate from the desktop-only
  editor-zoom control (`adjustEditorZoom()`). Opens from the bottom nav's
  "Text" tab.
- **One-at-a-time enforcement**: `mobileCloseEverything(exceptId)` is the
  single source of truth for what's open; `updateNavActiveState()` derives
  the bottom nav's highlighted/raised item from *actual* open state, not
  from what was last tapped.
- **Six former floating windows → bottom sheets** (Notepad, Draft Pad,
  Changelog, Proofreader, Send to Sammy, Stats), all at `66dvh`, deliberately
  **not** full-screen — Aaron was explicit that full-screen felt
  disorienting. Geometry save/restore (`save*State`/`restore*State`) is
  skipped entirely in mobile mode so a phone session can never corrupt
  stored desktop window positions — verified by seeding known desktop rects
  and confirming byte-identical after a full mobile session.
- **Guidance cards**: one icon-only rail strip on the right edge (not two
  rails — that was an earlier iteration, consolidated after review), 44×44
  squares, drawers open to the left reusing the same float-window shells
  desktop uses (content node is *moved*, never duplicated — same pattern
  `floatGuidanceCard`/`snapGuidanceCardHome` always used).
- **Landing nav**: one shared cluster (`#landing-nav-cluster`) used
  identically on desktop and mobile — no more separate desktop-radial /
  mobile-reflow. WHERE I LEFT OFF full-width hero; LAST COMPLETED / NEXT
  UNFINISHED flank a circular SYNC button in the empty square between them
  (this circle vs. true-50%-halves is a flagged geometric tradeoff, see
  "Landing Nav Redesign" entry).
- **Scene view**: the green status bar is now a fixed top-layer "control
  arm" in the upper-left (pill + Sammy/Draft-Log/Save stacked below it),
  editor content scrolls beneath it. Status pill is 88×44 (double the
  Sammy button's width, same height — this was inverted once and corrected,
  see the last two CHANGELOG entries if the dimensions ever look wrong
  again).
- **Proofreader "go to" arrow** (newest feature, "fourth review" entry):
  jumps from an issue to its location in the draft and highlights it via
  the CSS Custom Highlight API (`::highlight(proofreader-target)`), with a
  text-selection fallback. **Deliberately never mutates `editor.innerHTML`**
  — a wrapper `<span>` would get autosaved into Erica's actual prose. If
  you touch this, re-verify byte-identical `innerHTML` before/after, both
  code paths.
- **Library review markers**: appear at the *end* of an entry title / the
  *start* of a category title (this was a specific, deliberate placement
  request). Tapping one shows the reason; tapping *anything else* clears it
  permanently (persisted in `localStorage['bigtiff-library-review-dismissed']`),
  and it only reappears if Sammy sends a genuinely new
  `needsReviewReason` (see `reconcileLibraryStatuses`).

## Accessibility audit method (reuse this, it already caught real bugs)

Every round ran a script that walks all visible elements and checks (a)
computed font-size ≥12px, (b) WCAG AA contrast — **compositing every
translucent background layer from the element up**, including the
element's *own* background (an earlier, simpler version of this script
scored translucent chips against the wrong backdrop and missed a real
4.19:1 failure — see "Scene View Redesign" entry), and (c) touch targets
≥44×44 for interactive elements. Re-run this shape of check after any
future visual change; don't eyeball contrast.

---

## What is NOT verified — say this plainly to Aaron/Erica, don't let it get lost

**Nothing in this entire branch has been tested on real iOS Safari or a
real iPhone.** Every round was verified in desktop Chromium at the target
viewport sizes via `setMobilePreview()`. Specifically unconfirmed on real
hardware:
- Whether `(hover: none) and (pointer: coarse)` actually fires as expected
  on her iPhone (the query was confirmed to *parse* validly; it was never
  observed *matching*, since this dev environment is mouse-only).
- `contenteditable` caret visibility and scroll-into-view with the
  on-screen keyboard up (historically buggy on iOS specifically).
- `env(safe-area-inset-*)` behavior around the Dynamic Island / home
  indicator.
- Real Safari Page Zoom (only its geometric equivalent — a narrower
  viewport — was tested).
- Add to Home Screen / standalone launch (manifest + apple-touch-icon meta
  tags are in place from the first mobile-support commit; installation and
  standalone rendering were never exercised on a device).
- `CSS.highlights` / Custom Highlight API support (needs Safari 17.2+; the
  proofreader go-to arrow falls back gracefully if unsupported, but the
  fallback's *feel* under a real finger wasn't checked).
- Real touch gesture behavior generally (taps were simulated as click
  events, not real touch event sequences).

**This should be the next priority**: get it in front of Erica on the
actual device and report back what breaks. Everything above is a plausible
place for iOS-specific surprises even though the logic and layout are
verified correct in a standards-compliant browser.

---

## Suggested next steps

1. Real-device pass on Erica's iPhone 16 Pro Max — portrait, landscape,
   rotation, keyboard-up typing, Add to Home Screen.
2. If it holds up, promotion to `bigtiffsworld.com/app` (separate repo,
   separate manual step, ask first).
3. Anything flagged "not settled" in the CHANGELOG is Aaron's call, not a
   default to silently pick — surface it, don't resolve it quietly.
