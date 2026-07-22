# Big Tiff StoryForge — Claude Code Handoff & Project Brief

## For: Claude Code instance setting up and continuing development
## From: Claude (web session, 2026-06-22), at Aaron's direction
## Date: 2026-06-22

---

## What this project is

A single-file web-based writing tool (`writing.html`) for Erica (Aaron's
girlfriend, who has ADD) to write a children's/YA fantasy novel scene by
scene. The tool is a gift — every design choice should reduce friction and
cognitive load for her, not add steps or decisions.

A Grok-based story agent named "Sammy" writes scene guidance (prompts,
structure, canon) to JSON files on GitHub. The tool reads that data and
renders it as a structured writing environment. The tool never writes back
to Sammy's files. Erica's own work (drafts, status decisions) is persisted
in localStorage, layered on top of Sammy's canon but kept separate.

**Repo:** `https://github.com/footeprint-prog/big-tiff-storyforge`
**Story canon:** Google Drive, "Big Tiff" folder (Sammy's domain)
**Domain (new, for live deployment):** bigtiffsworld.com

---

## Architecture (critical — read before touching anything)

- **Single-file architecture.** The entire tool is one `writing.html` file:
  HTML + CSS + JS, no build step, no framework, no separate asset files.
  Tailwind CSS is loaded via CDN `<script>` tag. Font Awesome also via CDN.
- **Canon data sources (read-only, fetched live from GitHub on user click):**
  - `Big_Tiff_Scene_Structure.json` — scene prompts, statuses, structure
  - `Big_Tiff_Library_Entries.json` — story bible/lore entries by category
- **User data (localStorage, never sent anywhere):**
  - `bigtiff-scene-statuses` — per-scene user status decisions
  - `current-draft:<sceneId>` — per-scene draft autosave (HTML)
  - `bigtiff-global-changelog` — append-only audit log of sync-detected changes
  - `bigtiff-library-last-good-sync` — three-tier Library fallback cache
  - Various window-position state keys for floating panels

---

## What's already built and verified (as of 2026-06-22)

All of the following were browser-tested with Playwright in the session that
produced this handoff. The authoritative file is:
`writing_claude-session-2026-06-22_VERIFIED.html`
(renamed for clarity; should be pushed to GitHub as `writing.html`).

### Core features
- **Dynamic outline panel** (left sidebar) — acts/chapters/scenes tree,
  collapsible, expansion state persists across re-renders (e.g. status
  changes don't collapse what the user had open)
- **Scene-level editor** with contenteditable, formatting toolbar (B/I/U/list),
  word count, auto-save per scene
- **Library panel** (right sidebar) — Sammy's canon entries + Erica's own
  custom entries, with three-tier fallback (live sync → last-good-sync
  from localStorage → original hardcoded bibleData)

### Status system
- **Custom dropdown widget** (not a native `<select>` — replaced due to
  native select always showing current value duplicated in its own open list).
  Button trigger + connected sliding menu, each option colored to match its
  outline badge. Current status always on top; only valid alternatives shown
  below (Review never appears as a pickable option unless it's the current
  state — tool sets it, only the user clears it via confirm dialog).
- **Review notes** displayed below Scene Summary when a scene is in Review
- **Confirm dialogs** for clearing review status and for leaving a scene
  still flagged for review

### Sync system
- **Sync with Canon button** fetches both JSON files from GitHub in one click
- **`reconcileSceneStatuses`** — Sammy's `status`/`needsReviewReason` fields
  are authoritative; user's local status decisions are restored for scenes
  Sammy didn't re-flag; first-seen scenes default to `review`
- **`reconcileLibraryStatuses`** — same pattern for Library entries
- **Dedup logic** prevents duplicate changelog entries on repeat syncs of
  unchanged data
- **Structural change detection** via top-level JSON metadata comparison

### Floating panels (all draggable, resizable, position-persisted)
- **Notepad** — general notes, not scene-scoped
- **Draft Pad** — per-scene draft snapshots (named checkpoints), restore/
  rename/delete with confirms
- **Global Changelog** — read-only audit log with All/Scenes/Library/
  Structure filter tabs, auto-populated during sync
- **Proofreader** — calls free LanguageTool API, shows flagged issues with
  individual Apply + Apply All buttons. Apply uses exact character offsets
  (right-to-left for Apply All). Honest about its limitations — free tier
  misses many grammar issues; named "Proofreader" not "Coach" deliberately.
- **Send to Sammy** — opens a window with pre-formatted instructions for
  Sammy (contextual editorial review request + Draft Protection clause) plus
  scene title, guidance summary, and draft text. Copy All button with
  feedback.

### Library features
- Library canon sync from `Big_Tiff_Library_Entries.json`
- Three-tier fallback (live → cached → hardcoded)
- "!" review markers on entry titles AND category headers (hover shows which
  entries changed)
- Last-sync timestamp display

### Outline features
- Shortened scene titles (strips en-dash suffix for compact display)
- Status badges at fixed 100px width, matching the editor's dropdown
- Changelog icon button next to Sync Canon button

### UI/Layout
- Editor toolbar: status dropdown → scene label on first row; B/I/U/list +
  word count on left of second row; auto-save text + save icon + Draft Log +
  Send to Sammy + Proofreader on right
- Changelog icon buttons in both Outline and Library panel headers
- DRAFTS and CHANGELOG text buttons removed from top app header (access now
  via editor toolbar and panel icon buttons)

### Critical CSS fix
- `.notepad.hidden { display: none !important; }` — prevents closed floating
  panels from invisibly intercepting clicks. All five panel-open functions
  properly remove the `hidden` class (not just mask with inline style).

---

## What's still open (genuinely unbuilt)

### Must-build for launch
- [ ] **Auto-expand to first relevant scene** (`unfinished` or `review`)
      on initial load
- [ ] **Hidden login system** for bigtiffsworld.com — Aaron wants the tool
      accessible via a hidden login button on the new domain
- [ ] **Deployment** — the tool needs to actually be served from
      bigtiffsworld.com rather than opened as a local file or raw GitHub URL

### Planned but not yet scoped for build
- [ ] **Full draft assembly function** — assemble all per-scene drafts into
      one manuscript document (for bringing to Claude for full editorial pass)
- [ ] **Mobile version** — current desktop-first layout (draggable floating
      windows, multi-panel) needs a real responsive/touch pass
- [ ] **Visual skinning** — Aaron is producing a fantasy-themed visual mockup
      separately; functional correctness takes priority until the mockup is
      ready
- [ ] **Inter-scene outline editing** — scope not yet defined; WARNING:
      scene reordering must stay DISABLED for the user (hard requirement) —
      if this implies user-facing reordering, that conflict needs explicit
      resolution first
- [ ] **Connector unit types** in the scene schema (sequel/setup/deepening/
      ellipsis) — proposed for bridging gaps between scenes in the novel
      structure; pending Sammy's review of the full outline proposal

### Parked / low priority
- [ ] `safeGetItem`/`safeSetItem` localStorage wrappers — probably
      unnecessary for Erica's real browser
- [ ] Review pass on `Erica-Questions-Writing-Tool.md` — many already
      answered by choices made since 2026-06-21

---

## Key decisions already made (don't re-litigate)

1. **Grok Projects are not used.** Sandboxed environments caused corruption.
   All agents use external refresh protocol (GitHub + Drive + "Refresh
   yourself" trigger).
2. **Sammy's JSON is read-only to the tool.** Tool persists user choices
   locally in localStorage, never writes back.
3. **Sammy decides if a change needs review, not the tool.**
4. **Review status: tool-only to set, user-only to clear.**
5. **The "Writing Coach" was explored, built, tested, renamed to
   "Proofreader", and kept** — but with explicit acknowledgment that
   LanguageTool's free tier has real detection gaps. It's a mechanical
   proofreading pass, not a substitute for Sammy's contextual review or
   Claude's full-manuscript editing.
6. **Split editorial roles by job:** Sammy keeps generative/day-to-day
   story & canon work; Claude does the heavy editorial pass on the full
   assembled manuscript.
7. **Aaron plans to end the SuperGrok subscription** once the tool is
   complete. Sammy's role may migrate to free-tier Grok (Custom
   Instructions instead of Custom Agents) or potentially to Claude.

---

## Files in the repo (what each one is for)

| File | Purpose |
|---|---|
| `writing.html` | The tool itself — single-file, everything |
| `Big_Tiff_Scene_Structure.json` | Scene prompts/structure (Sammy writes) |
| `Big_Tiff_Library_Entries.json` | Story bible/lore entries (Sammy writes) |
| `AGENTS.md` | Fast orientation — read FIRST before any other doc |
| `CHECKLIST.md` | Development checklist + open items + parked ideas |
| `CHANGELOG.md` | Verified change log (only logged after real testing) |
| `WORKFLOW.md` | Active prescriptive agent workflow |
| `workflow-memory.md` | Historical lessons + incident log |
| `Project_Governance_and_Protocols.md` | Full governance rules |
| `Webtool_Editing_Protocol.md` | Safe editing methods |
| `Quick_Reference_Handoff.md` | Quick reference card |
| `OLD_Implementation_Checklist.md` | Superseded, kept for spec detail |
| `OLD_Webtool_Changelog.md` | Superseded, kept for historical detail |

---

## For the new Claude Code instance — immediate next steps

### Priority 1 — First things when work resumes

1. **Finalize the Focus button's actions.** The Focus button exists in the
   UI but its behavior is incomplete — needs debugging and full
   implementation of whatever actions it's meant to trigger. Aaron will
   clarify the intended behavior; Claude Code should examine the current
   `writing.html` for any existing Focus-related code/stubs and work from
   there.

2. **Create a landing page for the webtool.** This page appears AFTER login
   and BEFORE the full writing tool is shown. Should present beginning
   navigation options (not just dump Erica directly into the editor).
   Serves as a calm, oriented entry point — consistent with the ADD-friendly
   "one thing at a time" design philosophy.

### Priority 2 — Core functional additions

3. **Make the upper-right progress tracker functional.** The UI element
   exists but doesn't currently track or display real progress data. Needs
   to be wired to actual scene/draft completion state.

4. **Populate the stats window (upper right).** Connected to the progress
   tracker — should show meaningful writing statistics (scenes completed,
   word counts, etc.) with real data, not placeholder values.

5. **Add achievements, rewards (stickers?), and encouraging messaging.**
   Gamification/positive-reinforcement layer for Erica — stickers,
   milestone celebrations, gentle encouragement. Should feel warm and
   genuine, not patronizing. This is directly relevant to the ADD-friendly
   design goal: external motivation structures help when internal motivation
   is inconsistent.

6. **Design work: finalizing and applying the graphic skin.** Aaron is
   producing a fantasy-themed visual mockup separately. Once the mockup is
   complete, it will be broken down into individual skinnable elements and
   applied to the existing functional tool. Functional correctness takes
   priority until the mockup is ready — don't start this until Aaron
   provides the mockup assets.

### Priority 3 — Infrastructure and deployment

7. **Clone the repo** and confirm `writing.html` matches the verified
   version from this session (3835 lines, all features listed above
   present). If a different Claude instance has pushed changes since this
   handoff, the authoritative version is
   `writing_claude-session-2026-06-22_VERIFIED.html`.

8. **Read `AGENTS.md` first** — it's short and front-loads the things that
   are easy to get wrong.

9. **Set up deployment for bigtiffsworld.com** — Aaron wants the tool live
   and accessible via a hidden login button on this domain. This is likely
   a near-term task: getting the single-file tool served from a real URL
   with some form of simple authentication gating access.

10. **Set up git push capability** — this is the main advantage of Claude
    Code over the web chat: direct commit/push to the repo without Aaron
    manually downloading and uploading files each time.

### Priority 4 — Remaining open items from earlier sessions

11. **Auto-expand to first relevant scene** (`unfinished` or `review`)
    on initial load — small, self-contained task.

12. **Full draft assembly function** — assemble all per-scene drafts into
    one manuscript document for Claude editorial review.

13. **Mobile version** — responsive/touch pass for the current desktop-first
    layout.

14. **Inter-scene outline editing** — scope not yet defined; scene
    reordering must stay DISABLED (hard requirement).

15. **Connector unit types in schema** — pending Sammy's review of the
    proposed full outline.

16. **Refer to the 63 Playwright test scripts** (packaged as
    `bigtiff_playwright_tests_2026-06-22.zip`) for regression testing
    patterns. Key ones: `test_proofreader_full.py`,
    `test_all_panels_still_work.py`, `test_custom_dropdown_full.py`,
    `test_fix_repeated_word.py`.

---

## Important context about this tool's user

Erica has ADD. The tool's entire UX philosophy is:
- **Flow over features.** Don't add steps, decisions, or distractions.
- **Calm over correct.** If a choice is between a more technically "right"
  flow and a calmer one, prefer calmer.
- **One thing at a time.** Floating windows exist so she can pull up context
  when she needs it and dismiss it when she doesn't — not so everything is
  visible simultaneously.
- **Her writing is sacred.** Draft Protection is a real, enforced rule, not
  a suggestion. No automated process should ever overwrite or silently
  modify her draft text.
