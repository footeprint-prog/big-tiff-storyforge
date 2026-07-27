# Big Tiff StoryForge — Mobile Port Handoff

## For: whoever picks up mobile work next (agent or human)
## From: Claude sessions, 2026-07-24 – 2026-07-27, at Aaron's direction
## Status: **fully caught up as of 2026-07-27 — dev repo `main`, the
##         `claude/mobile-port` branch, and the live site all match.**
##         PR #6 (mobile UI rounds through 2026-07-26 + the achievements
##         engine) merged into `main` 2026-07-27, and a follow-up governance
##         doc pass (Sammy guardrails extracted into dedicated files, Dom
##         retired from Project_Governance_and_Protocols.md) rode in the
##         same branch/PR. Live site was already re-promoted 2026-07-26
##         (commit `c071209` in `big_tiff_launchpage`), verified
##         byte-identical against the actual domain. **CORRECTED 2026-07-27:**
##         every mobile UI round is in fact tested on a real device — Aaron
##         confirmed directly he checks all changes on a real iPhone 15 Pro,
##         iOS Safari, as a normal part of his workflow ("that is exactly
##         what I test everything on"). The "What is NOT verified" section
##         below had been carried forward with the wrong framing (implying
##         NO real hardware had ever touched this) across several sessions —
##         that framing is retired. What's still genuinely open: Aaron's own
##         test device (15 Pro) differs from Erica's phone (16 Pro Max per
##         her user profile) — a device-specific bug on her exact model
##         isn't ruled out just because Aaron's 15 Pro checks look fine.
## Dev branch: `claude/mobile-port` — long-lived, kept around, not deleted,
##             still the working branch for ongoing mobile UI iteration
##             (its GitHub Pages config is what gives the ~1min preview
##             rebuild). Merged to `main` periodically, not abandoned.
## Dev repo main: https://github.com/footeprint-prog/big-tiff-storyforge — **now matches `claude/mobile-port` exactly** (merge commit `dd9c6f3`, 2026-07-27). Future mobile-port commits will again lead main until the next merge — that's expected, re-sync when it's next convenient, not a problem to fix immediately.
## Live preview (dev repo Pages): https://footeprint-prog.github.io/big-tiff-storyforge/writing.html (tracks `claude/mobile-port`)
## **Real live site: https://bigtiffsworld.com/app/ — matches `main`/`claude/mobile-port` as of 2026-07-27.**

---

## Read this first, in this order

1. `AGENTS.md` — hard rules for this repo generally (don't re-derive).
2. This document, in full.
3. `CHANGELOG.md` — entries dated 2026-07-24 through 2026-07-26, newest
   first. Each has a **Verification notes** section stating exactly what
   was tested and how, and most have a **flagged, not settled** or
   **intentional exception** callout — read those before assuming
   something is a bug. The 2026-07-26 entry titled "Achievements & Usage
   Tracking (data layer)" is a separate subsystem from the mobile
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

## Deployment facts — three separate places, keep them straight

| Where | Repo | What's there |
|---|---|---|
| Dev repo Pages preview | `big-tiff-storyforge`, branch `claude/mobile-port` (Pages config) | **Current** — has every round through 2026-07-26 (rail collapse, drag-to-switch, circular status, achievements engine), rebuilds ~1 min after any push to that branch |
| Dev repo `main` | `big-tiff-storyforge` | **Merged and current as of 2026-07-27** (PR #6, merge commit `dd9c6f3`) — has everything: the 2026-07-25 mobile port, all 2026-07-26 mobile UI rounds, the achievements/tracking engine, and the 2026-07-27 governance/Sammy-extraction doc pass. `main` and `claude/mobile-port` are identical at time of writing; expect `main` to trail again once new commits land on the branch — re-sync via PR when convenient, it's routine, not urgent. |
| **Real live site** | `big_tiff_launchpage`, branch `main`, custom domain `bigtiffsworld.com` via Porkbun DNS (no Cloudflare in the path) | **Re-promoted 2026-07-26** (commit `c071209`) — has everything through the achievements engine. Assets/manifest were unchanged that round (checksums verified identical), only `app/index.html` needed copying. The 2026-07-27 governance-docs-only commits don't affect `writing.html`, so the live site's code is still current with no further promotion needed for those. Any future promotion (once `writing.html` itself changes again) is still a separate manual step Aaron requests explicitly — see the iteration-loop rule below. |

**The 2026-07-25 promotion already happened** the same way described in
prior versions of this doc: a direct byte-verified file copy of
`writing.html` → `app/index.html` plus the icon/manifest assets, pushed
straight to `big_tiff_launchpage`'s `main` (commit `7c3a047`), confirmed
live via direct requests to `bigtiffsworld.com`.

**One non-obvious fix required at promotion time, worth knowing if you ever
re-promote:** `manifest.webmanifest`'s `start_url` is `./writing.html` in
the dev repo (correct there) but must become `./index.html` for the live
site, since that's the file's actual name there. Skipping this makes Add to
Home Screen launch to a 404. Check `app/manifest.webmanifest`'s
`start_url` after any future re-promotion.

**If you need to promote again:** repeat the byte-identical-copy pattern
from `big-tiff-storyforge@main` (make sure main actually has what you think
it has first — see the table above) into `big_tiff_launchpage/app/`,
re-apply the `start_url` fix, and verify against the live domain directly
afterward — don't trust the Pages API alone, GitHub's raw-content CDN can
serve a stale cached copy for a bit even after a real deploy completes.

**Fast-iteration rule established 2026-07-25, still in force:** for
ongoing small mobile UI tweaks, the loop is commit + push to
`claude/mobile-port` only (auto-rebuilds the Pages preview in ~1 min) and
hand Aaron the preview URL. **Never** touch `bigtiffsworld.com` or
`big_tiff_launchpage` as part of that loop — promotion there is a separate,
explicit, later step only after Aaron has approved changes on the preview
URL.

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

### Scene-status control arm — redesigned twice since the original port
The original port made the green status bar a fixed "control arm" in the
upper-left with a pill + Sammy/Draft-Log/Save stacked below it. **That pill
is gone.** As of 2026-07-25/26 it's a **circular status button**:
- 56px circle, single letter code (**R**eview / **UF** unfinished /
  **C**omplete), 2px gold ring. Tapping it slides the alternate status
  option(s) out **horizontally** to the right (not the old vertical
  drop-down) — same status-picking rules as always (current status
  excluded, Review only offered if already active), just a sideways reveal
  since the circle sits flush against the screen's left edge.
- The Sammy/Draft-Log/Save action buttons now live behind a **separate
  dropdown-arrow tab** (`#status-actions-toggle`) below the circle,
  collapsed by default on every scene load. Wrapped in their own
  `.status-actions-bank` div, deliberately NOT the same flex row as
  `#auto-save-status` — collapsing the bank must never also hide the
  auto-save indicator (it's `position:fixed`, and `display:none` on an
  ancestor hides a fixed descendant regardless of its own positioning).
- The whole arm is a compound shape hanging flush off the header (no top
  border of its own — the header's own 2px gold bottom border, shared with
  desktop, caps it) with rounding only at the lower-right.
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
  chrome-tier 30px) at the top of the strip toggles `.rail-collapsed`,
  hiding all six icons down to just that arrow when docked.
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
