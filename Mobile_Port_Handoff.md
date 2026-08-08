# Big Tiff StoryForge — Mobile Port Handoff

## For: whoever picks up mobile work next (agent or human)
## From: Claude sessions, 2026-07-24 – 2026-08-04, at Aaron's/Erica's direction
## Status: **fully caught up as of 2026-08-04 — dev repo `main`,
##         `claude/mobile-port`, and the live site all match again**
##         (`claude/mobile-port` commit `92c83a8`, `main` commit `2546cdc`
##         via merge, `big_tiff_launchpage` `main` commit `706a79d`).
##         Two rounds landed 2026-08-04, both promoted live same-day:
##         1. **Desktop parity pass** - a line-item comparison of every
##            mobile-port change against desktop's actual behavior (most
##            were already shared code or deliberate touch-vs-mouse
##            differences, no action needed) surfaced four real gaps, all
##            fixed: the Review-pill WCAG contrast failure (only ever
##            patched mobile-only, desktop had carried the failing color
##            the whole time), Copy/Paste restored to the desktop editor
##            toolbar (an earlier "mobile-only" round had accidentally
##            deleted them from shared markup), the global text-size
##            control given a desktop entry point, and Library review-
##            marker click interaction extended to desktop (adds to, not
##            replaces, the existing hover tooltip). Also fixed in passing:
##            Stats/Text Size windows silently failing to open on the very
##            first tap each session, on **both** layouts - see "Desktop
##            parity pass" section below.
##         2. **Notepad/Draft Pad functionality review** - prompted by a
##            report that desktop Notepad "cannot type/add notes/add
##            library entry." Extensive real-interaction testing could not
##            reproduce any break in either feature, but found and fixed a
##            real viewer-role gating bug running in both directions - see
##            "Notepad/Draft Pad review" section below before assuming
##            either feature is broken again without first testing it
##            yourself the same way.
##         The 2026-08-03 round before these (keyboard-behavior revert,
##         Notepad-toggle-guard removal, Library review-marker fixes, Focus
##         button resize) and all prior rounds are unchanged and still
##         summarized below. See CHANGELOG for full round-by-round detail -
##         not repeated here.
## Live site (`bigtiffsworld.com`) status: **promoted 2026-08-04** (commit
##         `706a79d` in `big_tiff_launchpage`, copied from dev repo commit
##         `2546cdc`), verified **byte-identical** against the actual live
##         domain response (not just the Pages build API, which can report
##         "built" before a CDN edge actually serves the new bytes -
##         fetched `https://bigtiffsworld.com/app/` directly and diffed
##         against `git show HEAD:app/index.html` after stripping `\r`
##         from both sides, since this Windows checkout's `core.autocrlf`
##         makes the local working-tree copy differ byte-for-byte from the
##         LF-only git blob/live response even when the content is
##         identical - don't mistake that for a real diff next time).
##         Assets/manifest unchanged this round (checksums verified
##         identical); manifest's `start_url` was already correctly
##         `./index.html` from a prior promotion, nothing to fix this time.
## Real-device testing: every mobile UI round continues to get checked by
##         Aaron on a real iPhone 15 Pro, iOS Safari, as a normal part of his
##         workflow. What's still genuinely open: his test device (15 Pro)
##         differs from Erica's phone (16 Pro Max per her user profile) — a
##         device-specific bug on her exact model isn't ruled out just
##         because his 15 Pro checks look fine. The 2026-08-03 round's
##         Library review-marker bug was specifically device/browser-context
##         dependent (see below) - a reminder that "works on my test device"
##         doesn't rule out a different real-world context still failing.
##         The 2026-08-04 Notepad report couldn't be reproduced at all
##         despite thorough testing - if it recurs, get exact repro steps
##         (browser, logged in or guest, exact symptom) before assuming
##         it's the same root cause as anything documented here.
## Dev branch: `claude/mobile-port` — long-lived, kept around, not deleted,
##             still the working branch for ongoing mobile UI iteration
##             (its GitHub Pages config is what gives the ~1min preview
##             rebuild). Merged to `main` periodically, not abandoned.
## Dev repo main: https://github.com/footeprint-prog/big-tiff-storyforge — **matches `claude/mobile-port` exactly** (merge commit `2546cdc`, 2026-08-04). Expect `main` to trail again once new commits land on the branch - that's normal, re-sync before the next promotion.
## Live preview (dev repo Pages): https://footeprint-prog.github.io/big-tiff-storyforge/writing.html (tracks `claude/mobile-port`)
## **Real live site: https://bigtiffsworld.com/app/ — matches `main`/`claude/mobile-port` as of 2026-08-04, verified against the live domain directly.**

## Library review markers — a debugging story worth reading before touching this code
This round involved a long, winding investigation into Library review
markers (the exclamation-circle icon on entries/categories Sammy has
flagged) that's worth the full story rather than just the final diff,
because the actual root causes were **not** where they first appeared to be:
1. **Real bug found and fixed**: `armReviewClear()` attached a document-wide
   capture-phase click listener the instant a review marker was tapped to
   view its reason - the *next click anywhere in the app*, unrelated to
   that entry, silently and permanently dismissed the flag. This is what
   looked like "markers disappearing every time I refresh to test the next
   step" - continuing to use the tool after ever tapping a marker erased
   it. Fixed by reinstating the same tap-to-view/tap-away-to-dismiss
   design (Erica explicitly wanted this exact interaction kept, not a
   button-based alternative) - the fix was ensuring the dismiss action
   replaces the marker with the default white dot instead of leaving a gap,
   not changing the interaction model itself.
2. **A second real (smaller) bug**: category-level markers were also
   individually tappable/dismissible, which wasn't intended - only
   entry-level markers should be tap-to-view/tap-away-to-clear. Fixed by
   making category markers a plain, non-interactive visual indicator
   (`aria-hidden`, no `onclick`/`role`/`tabindex`) - tapping one now just
   falls through to the category header's normal expand/collapse.
3. **A false alarm that ate significant time**: markers appeared to work
   in Chrome and via the Home Screen icon but not in a specific regular
   Safari tab, even after a full sync reported success and even after
   closing/reopening that tab. This looked exactly like a Safari-specific
   code bug (and was treated as one for a round - the white-dot/marker-
   front treatment was reverted as a suspected cause, then confirmed
   NOT the cause and re-added). **The actual cause: stale `localStorage`
   in that one specific regular Safari tab** (almost certainly accumulated
   dismissed-review entries or an old cached library fallback from earlier
   test rounds) - a fresh **private-browsing** tab on the same device, same
   code, rendered correctly immediately. Lesson for next time a report
   says "works in Chrome/one browser context but not Safari specifically,
   even after reload": test a private/incognito tab on the failing browser
   *before* suspecting the code - it isolates "stale local storage on this
   one tab" from "actually broken in this browser" in about ten seconds,
   far faster than the code-level investigation this round went through
   first. The regular Safari tab's stale storage was left for the user to
   clear herself (Settings > Safari > Advanced > Website Data > remove the
   site entry) rather than fixed from this end - there's no code-side fix
   for one browser tab's accumulated local state.

## Desktop parity pass (2026-08-04) — where to find it, what's left
A full side-by-side comparison of every mobile-port change against
desktop's actual current code (not changelog claims) was published as an
HTML artifact during the session that did this work - it isn't saved to
this repo, so if you need the full 30+-item table again, it would need
regenerating rather than looked up. The short version: **most mobile-port
changes need zero desktop action** - they're either already shared code
(desktop gets them for free, no `isMobileMode()` gate exists) or
deliberate touch-vs-mouse differences (bottom nav, Focus button, sheet
chrome, guidance rail) that have no sensible desktop translation. Four
items were genuine gaps and are now fixed (contrast, Copy/Paste, text-
size control, review-marker click - see CHANGELOG's 2026-08-04 entry for
detail). **Before assuming a new mobile feature needs "porting to
desktop," check whether the underlying function is already
`isMobileMode()`-gated at all** - if it isn't, desktop already has it.

## Notepad/Draft Pad review (2026-08-04) — what was and wasn't found
A desktop bug report ("cannot type, cannot add new notes, cannot add
library entry") triggered an exhaustive functional review of both
features. Worth recording precisely what was and wasn't verified, since
the report couldn't be reproduced:
- **Tested and confirmed working**, via real `.click()` dispatch on the
  actual DOM buttons (not calling internal functions directly) plus real
  text entry (`execCommand('insertText')`/input events, not just setting
  `.value`): Notepad's New/type-title/type-content/Save/Add-to-Library,
  and Draft Pad's New/Rename/Load-into-Editor/Delete snapshot. Tested on
  both the local dev preview and the live production site.
- **Button geometry verified clean**: size, position, z-index, and
  `elementFromPoint()` overlap checked for every relevant button - nothing
  invisible, zero-sized, or covered by something else that `.click()`
  would silently bypass.
- **One false alarm**: "Load into Editor" appeared to no-op in testing -
  the function calls native `confirm()`, which this headless test
  environment auto-dismisses. Not a real bug; overriding `window.confirm`
  to return `true` confirmed the rest of the function works correctly.
- **What WAS found and fixed**: a `.primary-only`/viewer-role gating bug,
  in both directions at once. Desktop's New/Save buttons were fully
  blocked for `viewer` accounts, contradicting the documented spec
  (viewers should be able to create/edit notes, just not delete or touch
  the Library - see `project_persistence_architecture` memory / commit
  history for the spec). Mobile's relocated Delete/Add-to-Library buttons
  (`#m-notepad-editor-actions`) had the opposite problem - no gating at
  all, letting viewers delete and touch the Library from mobile. Both
  fixed to match spec on both layouts. **Confirmed this doesn't explain
  Erica's report** - her account (`ericap`) is `role: primary` in
  `accounts.json`, unrestricted either way.
- **If this report recurs**: get exact repro steps before assuming it's
  the same thing - which browser, logged in or guest, and what literally
  happens (blank area? console error? button does nothing at all?). A
  full re-test of both features from scratch, the way this round did it,
  is the fastest way to re-confirm or rule out a regression, but it
  couldn't find anything wrong this time and shouldn't be assumed broken
  again without new evidence.

---

## Read this first, in this order

1. `AGENTS.md` — hard rules for this repo generally (don't re-derive).
2. This document, in full.
3. `CHANGELOG.md` — newest first; entries run from 2026-07-24 through
   2026-08-05 now (a long run of mobile-only rounds, see the Status header
   above for the summary). Each has a **Verification notes** section
   stating exactly what was tested and how, and most have a **flagged, not
   settled** or **intentional exception** callout — read those before
   assuming something is a bug. The 2026-07-26 entry titled "Achievements &
   Usage Tracking (data layer)" is a separate subsystem from the mobile
   interaction work this doc covers — see the pointer section near the
   bottom of this doc.
4. `CHECKLIST.md` — "Mobile version of the tool" is marked done; a
   separate "Achievements — UI phase" item tracks what's left on the
   gamification side (icons, achievement book, trophy shelves — all
   presentation, the engine is done).
5. **Check whether Aaron has reported real-device findings since this doc
   was last touched.** If he has, those findings supersede the "not
   verified" section below — update it rather than trusting this file
   blindly. If you can't tell, ask rather than assume a device pass
   happened.

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
in this branch, across every round listed below. That check is not
optional for future changes either.

## How to test it yourself (no real phone required for CSS/logic work)

There's a `setMobilePreview(true|false)` function exposed on `window`,
callable from the console or `?forceMobile=1` in the URL. It flips the same
`isMobileMode()` the real media query drives, so it exercises the identical
code path. Pattern used throughout every round: force mobile mode, seed
`currentCanonData` with a fake scene or two if canon sync isn't available,
call `selectSceneFromCanon(scene)` directly, then measure real
`getBoundingClientRect()` / `getComputedStyle()` — never eyeball a
screenshot alone. Screenshots are also not renderable at all in some dev
environments (the Browser pane may not composite frames); when that's the
case, everything must be verified by measurement and by dispatching real
`PointerEvent`/`MouseEvent` sequences and checking resulting state, not by
looking at pixels.

Two real bugs were caught this way that a naive test would have missed:
- A drag-to-switch gesture's trailing `click` event would silently close
  the very drawer it had just switched to, unless explicitly swallowed
  (see "Guidance rail" below) — only caught by asserting state *after*
  firing a synthetic trailing click, not just after the drag itself.
- `full-outline-clear` unlocked on a brand-new project with zero scenes
  touched, because its original trigger read `unfinished` (scene default
  state) as "terminal" — only caught by testing the achievement against a
  freshly-seeded, untouched scene list, not just the "everything complete"
  happy path.
- (2026-07-27) The status circle's option menu positioned itself in the
  wrong place entirely — `left: calc(100% + var(--arm-pad-r) + 2px)` kept
  referencing a custom property (`--arm-pad-r`) that had been deleted
  earlier the same session. **An undefined custom property inside `calc()`
  invalidates the whole declaration**, not just that one value — so `left`
  silently fell back to a *different, unrelated* rule (the desktop
  default, `left: 0`) instead of erroring or doing nothing. This one
  **wasn't caught by the synthetic-event test suite** — the tests asserted
  state transitions (did the `open` class toggle, did the direction class
  apply) but never asserted the actual rendered position, so a completely
  wrong pixel position passed every check. Aaron caught it on the live
  preview. Lesson: when a change deletes a CSS custom property, grep for
  every other `var(--that-name)` reference before considering the change
  done — and add a position/rect assertion, not just a class-toggle
  assertion, to tests for anything that's positioned via CSS variables.

## Deployment facts — three separate places, keep them straight

| Where | Repo | What's there |
|---|---|---|
| Dev repo Pages preview | `big-tiff-storyforge`, branch `claude/mobile-port` (Pages config) | **Current** — `writing.html` content has everything through the 2026-08-04 Notepad/Draft-Pad viewer-role-gating round, commit `92c83a8`. Rebuilds ~1 min after any push to that branch. |
| Dev repo `main` | `big-tiff-storyforge` | **Merged and current as of 2026-08-04** (merge commit `2546cdc`). `main` and `claude/mobile-port` are identical at time of writing; expect `main` to trail again once new commits land on the branch. |
| **Real live site** | `big_tiff_launchpage`, branch `main`, custom domain `bigtiffsworld.com` via Porkbun DNS (no Cloudflare in the path) | **Promoted 2026-08-04** (commit `706a79d`, copied from dev repo commit `2546cdc`). Assets/manifest unchanged this round (checksums verified identical against the dev repo), only `app/index.html` needed copying; `start_url` was already `./index.html` from a prior round. Verified byte-identical against `https://bigtiffsworld.com/app/` directly (line-ending-normalized — this Windows checkout's `core.autocrlf` makes local/live diffs look nonzero even when content is identical; don't mistake that for drift). |

## Complete workflow: edit → verify → push → (optional) promote

Two local git clones, both already on disk — **edit these directly, never
re-clone or edit via GitHub's web UI/API:**
- `C:\Users\Erica\Desktop\AARON\STORY GIT\big-tiff-storyforge` (dev repo)
- `C:\Users\Erica\Desktop\AARON\STORY GIT\big_tiff_launchpage` (live site repo)

### Stage 1 — Standard edit loop (default; needs no special permission)
1. Edit `writing.html` directly in the `big-tiff-storyforge` clone, on
   branch `claude/mobile-port`.
2. Verify per "How to test it yourself" above — real measurement, real
   `PointerEvent` sequences for gestures, desktop re-checked unchanged.
3. `git add`, commit, `git push origin claude/mobile-port`. Pages
   auto-rebuilds the preview in ~1 min. Hand Aaron that URL.
4. **Stop here by default.** Stages 2 and 3 are separate, explicit,
   asked-for steps — never inferred from "ship this" alone.

### Stage 2 — Sync to `main` (only when explicitly asked, each time)
A "push to main" request is scoped to that one instance — don't treat it
as standing permission for future rounds, even later the same day.
1. `git checkout main && git pull origin main`
2. `git merge claude/mobile-port -m "<description>"` — should be clean; if
   it conflicts, stop and look, don't force.
3. Sanity-check the merged file has what you expect (grep for the
   specific fix/feature markers) before pushing — don't just trust a
   silent merge.
4. `git push origin main`, then `git checkout claude/mobile-port` to
   return to the working branch.

### Stage 3 — Promote to bigtiffsworld.com (only when explicitly asked)
The highest-stakes step — a real public production site.
1. **Make sure `main` actually has everything you think it has first**
   (run Stage 2 if not already done). Promoting stale/buggy `main`
   content ships that bug live — this has actually happened, always
   double-check rather than trusting a status line in this doc.
2. Byte-copy the file:
   `cp big-tiff-storyforge/writing.html big_tiff_launchpage/app/index.html`
3. Check whether assets/manifest need updating too — usually not:
   - Diff/checksum the dev repo's icon assets against
     `big_tiff_launchpage/app/assets/`; only copy if they've changed.
   - Check `big_tiff_launchpage/app/manifest.webmanifest`'s `start_url`
     is `./index.html` (**not** `./writing.html` — that's only correct in
     the dev repo, where the file is actually named `writing.html`;
     skipping this fix makes Add to Home Screen launch to a 404). Usually
     already correct from a prior promotion.
4. `cd` into `big_tiff_launchpage`, commit `app/index.html` (+ any changed
   assets) referencing the dev-repo commit it was copied from, `git push
   origin main`.
5. **Wait for the actual deploy**, don't fire-and-forget:
   `gh api repos/footeprint-prog/big_tiff_launchpage/pages/builds/latest`
   — poll (a few seconds apart, via an until-loop, not chained sleeps)
   until `status` is `"built"` and the commit SHA matches what you pushed.
6. **Verify against the real live domain directly** — never trust the
   Pages API alone, GitHub's CDN can serve stale content briefly even
   after a real deploy completes:
   `curl -s https://bigtiffsworld.com/app/ -o /tmp/live_check.html`, then
   diff byte-for-byte against what you pushed. This Windows checkout's
   `core.autocrlf` makes the local working-tree file differ from the
   LF-only live response even when content is identical — strip `\r`
   (`tr -d '\r'`) from both sides, or diff against `git show
   HEAD:app/index.html` instead of the working-tree file, before
   concluding there's a real difference. Also spot-check the manifest:
   `curl -s https://bigtiffsworld.com/app/manifest.webmanifest`.
7. Update this doc's Status header and the Deployment facts table above
   to reflect the new state (which commit is live, verified how), and
   sync that doc update back through `claude/mobile-port` → `main` too —
   so the docs don't immediately go stale relative to what they claim.

---

## Architecture established across this branch (don't rebuild, extend)

### From the original port (2026-07-24/25) — still true, unchanged
- **Detection:** `isMobileMode()` / `applyMobileLayoutClass()`. Toggling
  adds/removes `body.mobile-layout`; a `matchmedia` change listener plus
  `setMobilePreview()` both funnel through it.
- **Two-tier control sizing**, in `rem` so a UI text-scale change actually
  resizes controls: `--tap` (44px, anything touched while writing) and
  `--tap-sm` (30px, set-once chrome). A few **named, deliberate**
  exceptions below this floor exist — don't "fix" them without asking; see
  CHANGELOG for exact numbers and reasoning.
- **Runtime-measured CSS variables**, not hardcoded: `--mobile-header-h`,
  `--mobile-nav-measured` (kept **separate** from `--mobile-nav-h`, which
  the nav's own buttons consume — merging these created a real
  feedback-loop bug once, don't recombine them).
- **One-at-a-time enforcement**: `mobileCloseEverything(exceptId)` is the
  single source of truth for what's open.
- **Six former floating windows → bottom sheets**, all `66dvh`,
  deliberately not full-screen. Geometry save/restore is skipped entirely
  in mobile mode.
- **Proofreader "go to" arrow**: never mutates `editor.innerHTML` — uses
  the CSS Custom Highlight API with a text-selection fallback.

### Scene-status control arm — redesigned repeatedly since the original port
The original port made the green status bar a fixed "control arm" in the
upper-left with a pill + Sammy/Draft-Log/Save stacked below it. **That pill
is gone.** As of 2026-07-25/26 it's a **circular status button**, and as of
2026-07-27 it's also **draggable and corner-docking** rather than fixed:
- 56px circle, single letter code (**R**eview / **UF** unfinished /
  **C**omplete), 1px gold ring (thinned from 2px, 2026-07-27). Tapping it
  slides the alternate status option(s) out **horizontally** — same
  status-picking rules as always (current status excluded, Review only
  offered if already active).
- **Horizontally draggable along the header** (2026-07-27,
  `setupStatusBarDrag()`): a 300ms press-and-hold on the circle arms
  dragging; a plain tap still opens the status options. `top` stays pinned
  to the header (`var(--mobile-header-h)`); only `left` moves, clamped to
  the screen edges. Mirrors the guidance rail's own drag pattern. Position
  is inline-style-only, resets on login (`resetSceneStatusBarPosition()`,
  wired alongside the rail's own reset in `submitLogin()`).
- **Corner-morph docking** (2026-07-27, `updateStatusBarDock()`): release
  within 48px (0.5in) of a screen edge and the arm snaps flush and morphs —
  the wall-facing border and that bottom corner's rounding drop, the free
  side keeps both. Both bottom corners round when genuinely free-floating
  mid-header. Runs on drag release, every scene render, and login reset,
  so the shape always matches wherever the arm currently is. `#scene-
  status-bar`'s z-index (470) is deliberately above the guidance rail (45)
  and an open guidance drawer (460) — when the two draggable elements
  (this arm and the rail) end up overlapping in a corner, the status
  circle wins, per Aaron's spec.
- The status option menu opens **left or right, decided fresh at open time**
  (`positionStatusMenuDirection()`) from the option row's real
  `scrollWidth` against actual available viewport space — necessary once
  the arm could be anywhere along the header, not just flush-left. Its
  `left`/`right` offset depends on the arm's own padding + border (8px +
  2px = 10px) — **if the arm's padding or border ever change again, this
  needs to change with it**; there's no shared variable expressing that
  relationship after `--arm-pad-r` was removed (see the bug in "How to
  test it yourself" above) — worth reintroducing one if this drifts again.
- The Sammy/Draft-Log/Save action buttons live behind a **separate
  dropdown-arrow tab** (`#status-actions-toggle`) below the circle,
  collapsed by default on every scene load, centered under the arm.
  Wrapped in their own `.status-actions-bank` div, deliberately NOT the
  same flex row as `#auto-save-status` — collapsing the bank must never
  also hide the auto-save indicator (it's `position:fixed`, and
  `display:none` on an ancestor hides a fixed descendant regardless of its
  own positioning).
- The whole arm is a compound shape hanging flush off the header (no top
  border of its own — the header's own 2px gold bottom border, shared with
  desktop, caps it). Margins standardized to 8px throughout (bottom padding
  under the button stack is 14px specifically, to match the ~14-16px side
  clearance those buttons get from being centered in the wider,
  circle-driven column — see `BUTTON_STYLE_GUIDE.md`).
- `updateStatusSelector()` renders both a `.status-trigger-label` (full
  word) and `.status-trigger-code` (short code) per status option; mobile
  CSS shows the code and hides the label, desktop is the reverse — so the
  same JS serves both control shapes with no branching.

### Guidance rail — redesigned repeatedly since the original port
- Enlarged from 44px to 56px icons (2026-07-25/26), still one touching
  connected strip, not separate bordered buttons.
- **Draggable to reposition** (2026-07-25/26): a press-and-hold (currently
  300ms — was 1000ms initially, shortened after Aaron found it sluggish)
  arms dragging; a plain tap still opens a drawer. Position lives only in
  an inline style on the live element, never persisted, so a fresh page
  load (a real login) always starts centered — no explicit reset code
  needed for that case, though `submitLogin()` also calls
  `resetGuidanceRailPosition()` explicitly for the case where a different
  account logs in without a full reload.
- **Collapsible** (2026-07-26): a small arrow tab (`#mobile-rail-toggle`,
  chrome-tier 30px) toggles `.rail-collapsed`, hiding all six icons down to
  just that arrow when docked. **Moved from the top to the bottom of the
  strip (2026-07-27)** so it never ends up positioned under the (now
  draggable) scene-status arm, which lives up near the header — if the
  toggle is ever unreachable/hidden behind something, check this hasn't
  drifted back.
- Icon glyph size: **1rem, matching the editor-toolbar and status-button
  icons** (`.text-tool-btn`, `.control-bar-item`) — this was bumped to
  2.5rem for a few hours on 2026-07-27 to try to close up the visible
  margin around each icon inside its 56px box, then reverted the same day
  after Aaron flagged it as oversized and inconsistent. **The margin
  complaint itself is still open** (see CHANGELOG's 2026-07-27
  "Flagged, not settled") — don't re-attempt via icon size again without
  asking; a fix needs to come from somewhere else (box/padding geometry,
  not the glyph).
- **Drag-to-switch between drawers** (2026-07-26): once a drawer is
  already open, dragging on the rail switches which one is shown in real
  time (hit-tested per `pointermove` against each `.rail-tab`'s rect) —
  **this intentionally supersedes repositioning while a drawer is open.**
  Reposition only when idle with nothing open; two different jobs for one
  gesture, disambiguated by whether `mobileOpenDrawerKey` is set at the
  start of the press. Opening a drawer always force-expands the rail first
  (a collapsed rail would hide the very tabs needed to switch away from
  it). The trailing `click` after a scrub that actually switched tabs is
  swallowed the same way a reposition-drag's trailing click already was —
  without that, ending a scrub on a tab immediately closes it.
- **Copy-to-clipboard button** (2026-07-25) added to each of the six
  drawers' own header row, mobile-only, reading the already-rendered
  `innerText` so formatting matches what's on screen.
- Drawer header row (2026-07-26): switched from a 3-way
  `justify-content: space-between` (title / icon-cluster / close-X) — which
  stranded the copy button at the row's exact midpoint instead of next to
  the X — to `flex-start` with the title growing, clustering copy+X
  together on the right.

### Header (2026-07-26)
- `.fantasy-header` gained a 2px gold bottom border — **on both desktop
  and mobile**, per Aaron's explicit call, unlike almost everything else in
  this doc which is mobile-only.
- Mobile additionally gets a 4px gap below the header, implemented as a
  **margin** (not padding/height) specifically so it's excluded from the
  `getBoundingClientRect().height` that `--mobile-header-h` measures — the
  fixed status arm needs to stay flush under the gold border while flow
  content still gets the gap.

## Accessibility audit method (reuse this, it already caught real bugs)

Every round ran a script that walks all visible elements and checks (a)
computed font-size ≥12px, (b) WCAG AA contrast — compositing every
translucent background layer from the element up, including the element's
own background, and (c) touch targets ≥30×30 (chrome-tier) or ≥44×44
(primary-tier) for interactive elements. Re-run this shape of check after
any future visual change; don't eyeball contrast or tap sizes.

---

## Achievements & usage-tracking system — related but separate subsystem

A large data-layer addition landed 2026-07-26: 188 embedded achievements,
an encrypted `progress.json` synced alongside drafts/checkpoints/stats, a
writing-session concept, and ~25 named tracking hooks (several of them
mobile-specific: `mobileDrawerOpen`, mobile write-day tracking, etc.). This
is genuinely a different concern from the touch-interaction work above —
**see the CHANGELOG entry "Achievements & Usage Tracking (data layer)"
(2026-07-26) for the full architecture, and `CHECKLIST.md`'s "Achievements
— UI phase" item for what's left (icons, achievement book, trophy
shelves).** Not duplicated here to avoid the two docs drifting apart; if
you're doing mobile interaction work and need to know whether it should
also fire a tracking event, check that CHANGELOG entry for the
`trackEvent()` call-site pattern.

One item from that system is genuinely mobile-relevant and worth flagging
here directly: `full-outline-clear` is **human-confirmed**, not automatic —
it prompts Erica for confirmation (once every scene is Complete with no
review flags, at most once per day) rather than unlocking off scene status
alone. If you ever touch scene-completion flows on mobile, be aware this
confirmation dialog (`confirm()`, a blocking browser dialog) can appear.

---

## Real-device testing status (corrected 2026-07-27)

**Aaron confirmed directly (2026-07-27) that he tests every mobile round on
a real iPhone 15 Pro, iOS Safari, as standard practice** — the framing this
section used to carry ("nothing has ever touched real hardware / a real
finger") was wrong and had been repeated across multiple sessions without
being checked against him. Don't resurrect that framing.

What's still a genuine, narrower gap: Aaron's test device (15 Pro) isn't
necessarily Erica's own phone (16 Pro Max per her user profile) — a
device-specific quirk unique to her exact model/iOS version isn't ruled out
just because his 15 Pro checks look fine. If that distinction matters for a
specific bug report, ask which device it was seen on rather than assuming.
The items below are the specific technical behaviors worth double-checking
on whichever device is in hand when touching related code — not a claim
that they're all still unverified in the abstract:
- Whether `(hover: none) and (pointer: coarse)` fires as expected — worth a
  quick sanity check whenever detection logic changes.
- `contenteditable` caret visibility and scroll-into-view with the
  on-screen keyboard up (historically buggy on iOS specifically).
- `env(safe-area-inset-*)` behavior around the Dynamic Island / home
  indicator.
- Real Safari Page Zoom behavior at high zoom levels.
- Add to Home Screen / standalone launch.
- `CSS.highlights` / Custom Highlight API support (Safari 17.2+; the
  proofreader go-to arrow falls back gracefully if unsupported).
- **Feel/timing judgments** — the rail's and status-arm's press-and-hold
  thresholds (300ms) are dev-tool-verified via synthetic `PointerEvent`
  sequences for logic correctness; whether the exact delay *feels* right is
  a real-thumb judgment call each round, worth asking about specifically.
- The `confirm()` dialog used by the Full Outline Clear gate — native
  browser confirm dialogs can render inconsistently across mobile browsers;
  worth a specific look if that flow changes.

---

## Suggested next steps

1. **Read whatever Aaron reports back from any device pass first** —
   don't start new mobile work without checking whether something above
   just got confirmed broken or confirmed fine.
2. Fix whatever he finds. Small, isolated, measurement-verified — same
   discipline as every round in this branch.
3. Anything flagged "not settled" in the CHANGELOG is Aaron's call, not a
   default to silently pick — surface it, don't resolve it quietly.
4. **Any FUTURE promotion** needs the same manual byte-copy step
   described above — it does NOT auto-deploy from the dev repo, and being
   caught up as of 2026-07-26 doesn't change that going forward. Confirm
   with Aaron before promoting again; per the iteration-loop rule, that's
   an explicit later step, not something to do automatically just because
   commits have piled up.
5. The achievements UI phase (icons, achievement book, weekly trophy
   shelves) is the next planned body of work — see `CHECKLIST.md`.
