# Big Tiff StoryForge — Mobile Port Handoff

## For: whoever picks up mobile work next (agent or human)
## From: Claude session, 2026-07-24 – 2026-07-25, at Aaron's direction
## Status: **merged to main AND promoted to the live site.** Aaron is
##         actively testing on Erica's real iPhone right now — this doc
##         has not yet been updated with device-test findings, so treat
##         it as "should be right" rather than "confirmed right on device."
## Dev branch: `claude/mobile-port` — merged via PR #4, kept around, not deleted
## Dev repo main: https://github.com/footeprint-prog/big-tiff-storyforge (has it)
## Live preview (dev repo Pages): https://footeprint-prog.github.io/big-tiff-storyforge/writing.html
## **Real live site: https://bigtiffsworld.com/app/ — HAS THE MOBILE BUILD.**

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
5. **Check whether Aaron has reported real-device findings since this doc
   was last touched.** If he has, those findings supersede the "not
   verified" section below — update it rather than trusting this file
   blindly. If you can't tell, ask rather than assume the device pass went
   cleanly.

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

## Deployment facts — three separate places, keep them straight

| Where | Repo | What's there |
|---|---|---|
| Dev repo Pages preview | `big-tiff-storyforge`, branch `claude/mobile-port` (Pages config) | Mobile build, rebuilds ~1 min after any push to that branch |
| Dev repo `main` | `big-tiff-storyforge` | Mobile build, merged via PR #4 on 2026-07-25 |
| **Real live site** | `big_tiff_launchpage`, branch `main`, custom domain `bigtiffsworld.com` via Porkbun DNS (no Cloudflare in the path) | **Promoted 2026-07-25** — `app/index.html` now IS the mobile build |

**The promotion already happened.** `big_tiff_launchpage` is cloned locally
as a sibling of this repo (`../big_tiff_launchpage`). The promotion was a
direct file copy — `writing.html` → `app/index.html`, plus the three new
`app-icon-*.png` and `manifest.webmanifest` into `app/` — verified
byte-identical to the dev-repo source before committing (size match + PNG
signature check on the icons), then pushed straight to that repo's `main`
(commit `7c3a047`), and confirmed live via direct requests to
`bigtiffsworld.com` (not just the Pages API) plus a real-browser load with
zero console errors.

**One non-obvious fix required at promotion time, worth knowing if you ever
re-promote:** `manifest.webmanifest`'s `start_url` is `./writing.html` in
the dev repo (correct there) but had to become `./index.html` for the live
site, since that's the file's actual name there. If you copy files over
naively without this fix, Add to Home Screen will launch to a 404. Check
`app/manifest.webmanifest`'s `start_url` after any future re-promotion.

**If you need to promote again** (e.g. after further mobile fixes land):
repeat the same byte-identical-copy pattern from `big-tiff-storyforge@main`
into `big_tiff_launchpage/app/`, re-apply the `start_url` fix, and verify
against the live domain directly afterward — don't trust the Pages API
alone, GitHub's raw-content CDN can serve a stale cached copy for a bit
even after a real deploy completes (happened once during this promotion;
`git show <ref>:path` avoids it, `curl raw.githubusercontent.com` doesn't
always).

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

**Status as of 2026-07-25: Aaron is testing on Erica's real iPhone right
now, on the live site (`bigtiffsworld.com/app/`), which does have the
mobile build.** This doc has not yet been updated with what he finds — if
you're reading this after that testing session, look for a newer
CHANGELOG entry or ask before trusting the list below. Everything under
this heading was true *going into* that test pass, verified only in
desktop Chromium at target viewport sizes via `setMobilePreview()` — none
of it had been confirmed on real hardware as of the promotion:
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

Aaron was pointed at these specific things to check: Add to Home Screen +
standalone launch, caret/keyboard behavior while typing, rotation with a
scene open, Dynamic Island / home indicator clearance, the Proofreader
go-to arrow, and pinch-to-zoom. Everything above is a plausible place for
iOS-specific surprises even though the logic and layout are verified
correct in a standards-compliant browser.

---

## Suggested next steps

1. **Read whatever Aaron reports back from the device pass first** —
   don't start new mobile work without checking whether something above
   just got confirmed broken or confirmed fine. Update this doc's "not
   verified" list based on what he says, don't leave it stale.
2. Fix whatever he finds. Small, isolated, screenshot-verified — same
   discipline as every round in this branch (see CHANGELOG entries for the
   pattern: measure, don't eyeball, and re-run the full accessibility audit
   after any visual change).
3. Anything flagged "not settled" in the CHANGELOG is Aaron's call, not a
   default to silently pick — surface it, don't resolve it quietly.
4. Any further fix needs re-promoting to `bigtiffsworld.com` the same way
   (see Deployment facts above) — it does NOT auto-deploy from the dev
   repo, promotion is a separate manual copy every time.
