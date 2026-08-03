# Big Tiff StoryForge – Development Checklist

This checklist must be followed for every change made to the project.

## Pre-Change Checklist
- [ ] Read the latest `workflow-memory.md` (especially recent incident logs)
- [ ] Understand the current stable state of `writing.html`
- [ ] Plan the change in small, testable increments

## During Change Checklist
- [ ] Make changes in small, isolated steps
- [ ] Never perform large single-block replacements of HTML + JS together
- [ ] Use safe editing methods (precise string replacement or small targeted edits)

## Post-Change Verification Checklist
- [ ] Run: `python3 validate_html.py writing.html`
  - [ ] Confirm: Div balance passes
  - [ ] Confirm: Expected IDs are present (or note why they aren't)
- [ ] Run: `python3 verify_github_push.py` (when preparing to push)
- [ ] Manually test key functionality (ideally in an actual browser, not just by reading the code):
  - [ ] Panel toggles (Outline & Library buttons)
  - [ ] Side buttons remain visible after clicking
  - [ ] Scene selection from Outline populates guidance correctly
  - [ ] Editor formatting buttons work
  - [ ] Save Draft works
- [ ] Check browser console for errors (especially `SyntaxError` or `ReferenceError`)
- [ ] **"Done" means tested, not just written.** Do not report a feature as complete based on reading the code alone — confirm it actually runs as expected.

## Push Checklist (Mandatory)
- [ ] Read actual file content before pushing (never use placeholder strings)
- [ ] Commit with clear, descriptive messages
- [ ] After pushing, verify raw GitHub URLs contain real code:
  - https://raw.githubusercontent.com/footeprint-prog/big-tiff-storyforge/main/writing.html
  - https://raw.githubusercontent.com/footeprint-prog/big-tiff-storyforge/main/workflow-memory.md
- [ ] Confirm file sizes are reasonable (writing.html > 50KB)
- [ ] Confirm no placeholder text exists in pushed files

## Never Do
- Use placeholder strings when pushing to GitHub
- Make large untested replacements
- Skip verification steps
- Ignore console errors
- Report something as "done" or "verified" without actually checking it in this session

---

## Open Items Carried Forward From Original Spec (2026-06-16)

These were defined in the original, more detailed Implementation Checklist
(now archived as `OLD_Implementation_Checklist.md`) and confirmed via direct
code check on 2026-06-22 to still be **unbuilt**. Carried forward here so
they aren't lost now that the old document is marked historical.

- [x] **Global Outline/Lore Changelog** — draggable window, same style as
      Notepad/DraftPad, showing Sammy's canon + structure changes. DONE
      2026-06-22 — built with All/Scenes/Library/Structure filter tabs.
      Checklist was stale here; verified directly against the live file
      (`#changelog-window` exists) before correcting this entry.
- [x] **Library changelog** — separate from the Outline/lore changelog above;
      tracks changes specifically to `Big_Tiff_Library_Entries.json` content.
      (See `Big_Tiff_Standardized_Library_Entry_Template.md` Section 6.)
      DONE 2026-06-22 — verified against real Sammy-populated data, not just
      simulated test data. Library panel rendered correctly, Changelog
      "Library" filter showed entries, last-sync timestamp updated, no
      console errors. Also added: a small "!" marker before the title of
      any Library entry currently flagged for review by Sammy (hover shows
      the reason) — same signal driving the changelog.
- [x] **Library change detection + alert badge** — Library panel should show
      a visible indicator when canon-driven changes affect Library-sourced
      entries. DONE 2026-06-22 — implemented as a two-level "!" marker
      system rather than a single panel badge: each flagged entry's title
      shows a "!" with its review reason on hover, and each category header
      also shows a "!" (hover lists which entries inside are flagged) so the
      user can spot changes without expanding every category. Verified with
      a real browser test against entries with mixed flagged/unflagged
      status across multiple categories.
- [x] **Last sync timestamp in Library panel** — DONE 2026-06-22, built
      alongside Library canon sync itself (`#library-last-sync-time`
      verified present in the live file). Checklist was stale here too.
- [ ] **Auto-expand to first relevant scene** (`unfinished` or `review`) on
      initial load, rather than requiring the user to pick a scene manually
      every session.
- [ ] **Scene reordering must stay disabled** — confirmed NOT present in the
      live file as of 2026-06-22 (hard requirement, verified, no action
      needed unless this changes).
- [x] **Mobile version of the tool.** DONE 2026-07-24 — built as an
      interaction-model change for touch devices, not a responsive CSS pass.
      Detection is capability-based (`hover: none` + `pointer: coarse`), never
      width, so landscape at 956px cannot flip to desktop. Floating windows
      became full-screen sheets (one at a time); guidance cards became a
      right-wall rail of six vertical drawer tabs; a bottom nav carries
      Home/Outline/Library/Notepad/Focus. Full accessibility pass: no text
      below 12px, zero measured WCAG AA contrast failures, all touch targets
      ≥44×44. Desktop verified unchanged by direct measurement. See CHANGELOG
      for what was and was not verified. **CORRECTED 2026-07-27:** earlier
      versions of this entry said nothing was tested on real iOS Safari —
      Aaron confirmed directly that he tests every mobile round on a real
      iPhone 15 Pro as standard practice, so that framing was wrong and is
      retired (see `Mobile_Port_Handoff.md`'s "Real-device testing status"
      section). The narrower open point: his test device isn't necessarily
      Erica's own phone (16 Pro Max), so a quirk unique to her exact model
      isn't ruled out by his checks alone.
      **UPDATE 2026-07-26:** several further refinement rounds landed on
      `claude/mobile-port` (not yet re-promoted to bigtiffsworld.com — see
      `Mobile_Port_Handoff.md`'s Deployment facts table): the scene-status
      pill became a circular R/UF/C button with a horizontal slide-out
      option drawer and a collapsible action-button bank; the guidance rail
      gained drag-to-reposition, a collapse toggle, and drag-to-switch
      between open drawers; each guidance drawer gained a copy-to-clipboard
      button. `Mobile_Port_Handoff.md` was rewritten 2026-07-26 to cover all
      of this — read that doc, not just this entry, before further mobile
      work. Per the 2026-07-27 correction above, the drag/collapse/scrub
      gestures (including the scene-status arm's own drag, added
      2026-07-27) do get checked on Aaron's real iPhone 15 Pro each round —
      not an untouched-by-any-finger state.
      **UPDATE 2026-07-27:** the scene-status arm gained its own
      press-and-hold horizontal drag plus corner-morph docking (snaps flush
      to whichever screen edge it's released near, border/rounding morph
      to match), and its option menu now opens left or right depending on
      available space instead of always right. A same-day fix-up round
      corrected two real problems from the initial version (a positioning
      bug from a deleted CSS variable, and an icon-size change that
      overshot a "reduce the margin" request) — see CHANGELOG's 2026-07-27
      entry for full detail, including one item still flagged not settled
      (rail-icon margin). **UPDATE 2026-07-27 (later same day):** Aaron
      asked to push everything live; `main` was re-synced with
      `claude/mobile-port` (picking up the fix-up round, not just the
      buggy corner-morph round) and promoted to bigtiffsworld.com,
      verified byte-identical against the live domain directly. All three
      — `claude/mobile-port`, `main`, and the live site — match as of this
      update. See `Mobile_Port_Handoff.md`'s Status header for the
      current state if this has since drifted.
- [x] **Achievements & usage-tracking data layer.** DONE 2026-07-26 — 188
      achievements embedded with machine-readable rules (cross-checked
      against the source JSON, 0 mismatches), an AES-encrypted
      `progress.json` synced alongside drafts/checkpoints/stats (counters,
      per-day buckets, hour-of-day histogram, personal bests, per-scene
      records, streaks, unlock log), ~25 named `trackEvent()` call sites, a
      writing-session concept (15min idle timeout, resumes across a
      refresh), and the achievement-evaluation engine
      (`deriveFacts`/`evaluateAchievements`/`getAchievementBook`). Merge
      logic is a verified join-semilattice (commutative/associative/
      idempotent). `stats.json` bumped to schemaVersion 2, draining a
      2000-entry PLAINTEXT activity log that had been publishing when Erica
      writes to the public data repo — that leak predates this work and is
      now closed. `full-outline-clear` is human-confirmed (prompts once
      every scene is Complete with no review flags, at most once/day, per
      Aaron: "unfinished" is a scene's natural resting state, so no
      automatic reading of the outline can prove the book is actually
      done). See the CHANGELOG entries dated 2026-07-26 for full
      verification detail (boundary thresholds, streak edge cases, a
      sabotage test proving the analytics layer can never break the
      editor, write-amplification measurement of 1 commit per 120 typing
      events). **This is the engine only** — see the next item for the
      remaining presentation work.
- [ ] **Achievements — UI phase.** The data layer shipped 2026-07-26
      (188 achievements, encrypted `progress.json`, usage tracking, the
      engine, and the human-confirmed Full Outline Clear gate). Everything
      below is presentation; the engine already emits
      `bigtiff:achievement` / `bigtiff:progress-updated` events and
      `getAchievementBook()` returns unlocked state, percent, remaining and
      a formatted label for all 188, so none of this needs engine changes.
      Added 2026-07-26 at Aaron's direction.
      1. **Unique icons for each achievement.** Definitions already carry an
         `ic` placeholder name per achievement; the Stats teaser currently
         falls back to per-category FontAwesome icons. Needs a decision on
         art style, source, and storage (sprite / inline SVG / image files).
         **Discussion item first, not a build item.**
      2. **Achievement book (bank)** showing completed achievements.
      3. **Achievement window/list split** into achievements in progress
         (with status) vs. not yet accomplished, showing a
         completed/total count.
      4. **Weekly achievements included but identified separately** from the
         188 lifetime ones.
      5. **Desktop header trophy shelf.** A visible bank in the header with
         **5 empty slots** styled as empty trophy shelves — matching the 5
         weeklies drawn each week. Completing a weekly places a unique
         "live" animated creature GIF in a jar on that shelf. Two asset jobs:
         generate the shelf artwork, then generate a bank of animated
         creature GIFs. **Shelves empty on weekly reset.**
      6. **Mobile achievements nav button + window.** Weeklies shown in a
         lower-left collapsible bank of the same trophy shelf slots. Must
         respect the existing mobile patterns (`mobileCloseEverything()`
         one-at-a-time enforcement, `updateNavActiveState()` deriving
         highlight from real open state, ≥44px targets) and must not collide
         with the draggable guidance rail on the right edge.
      - **Blocker for items 4-6:** the weekly pool is incomplete. The JSON
        has 13 entries; Aaron is sourcing the rest toward 21. Selection
        logic is pool-size agnostic, so adding them is a data edit — but the
        weekly selection/reset itself still needs wiring (deterministic
        seeded pick of 5 per ISO week, identical on every device and not
        re-rollable by refreshing).
- [ ] **Visual skinning of the tool.** Aaron is producing a fantasy-themed
      visual mockup separately; once complete, it will be broken down into
      individual elements to "skin" the existing functional tool. Directly
      related to the already-parked "main writing area layout/visual
      polish" item below - this is the broader version of that same
      deferred work. Functional correctness takes priority until the
      mockup is ready. (Added 2026-06-22.)
- [ ] **Full draft assembly function.** *(Also gates the
      `manuscript-assembled` achievement, which ships unreachable until this
      exists — its `counter.assemblyRun` metric is already in place and sits
      at 0, so the achievement works the day this lands.)* A function to
      assemble all completed (or all, depending on design) per-scene drafts
      into one complete manuscript document. Context: Aaron's original plan was for
      the tool to assemble the complete draft, then bring that assembled
      document to Claude directly (outside this tool) for full-manuscript
      developmental/line editing - Grok has had difficulty with long text
      files, which is a strength of Claude's. This item is just the
      assembly function itself; the actual editing pass happens in a
      separate Claude conversation, not inside the tool. (Added 2026-06-22.)
- [ ] **Inter-scene outline editing function.** Some way to edit
      relationships/structure *between* scenes from within the tool
      (exact scope not yet defined - needs follow-up discussion to clarify
      what this covers: reordering, merging, splitting scenes, editing
      transitions, etc.). Note: scene reordering is currently a confirmed
      hard requirement to keep *disabled* for the user (see above) - if
      this new item implies user-facing reordering, that conflicts with
      the existing rule and needs to be resolved explicitly before building
      anything, not assumed either way. (Added 2026-06-22.)

## Parked Ideas / Not Now

A holding pen for things mentioned in passing that aren't urgent. Review
periodically; move items up into active work when ready, or strike them out
if no longer wanted.

- [ ] **Strengthen Sammy's specialized storytelling expertise.** Aaron wants
      to spend dedicated time with Sammy identifying additional foundational
      resources (craft references, structure frameworks, genre-specific
      material) to deepen his story-writing expertise beyond the current
      instruction set. Goal: be able to trust Sammy's editorial judgment more
      fully on scene creation and overall Big Tiff storyline decisions — this
      matters directly to the tool too, since the review-status workflow
      already treats Sammy's `needsReviewReason` calls as authoritative.
      Strengthening his foundation makes that trust better-placed. (2026-06-22)
      UPDATE 2026-06-22: Aaron plans an external knowledge document including
      complete ebook files on storytelling/craft. This requires Claude
      Projects or an equivalent persistent-knowledge-base feature (free-tier
      Claude can't retain uploaded files across sessions) — relevant if
      Sammy ever moves platforms. See resolved item below for the related
      platform/subscription discussion.

- [x] **RESOLVED 2026-06-22: Writing Coach (grammar) + Sammy-platform question.**
      Explored adding an in-tool grammar/tense checker (LanguageTool API).
      Built and shipped, but real-world testing showed the free tier's
      detection quality is materially weaker than its own website demo
      (confirmed via direct API response inspection: free tier returned
      zero matches on a paragraph with clear errors, while LanguageTool's
      own site caught 9). Premium pricing (~$5-7/mo) and the free tier's
      apparent instability (conflicting reports of shrinking limits) made
      paying for it a bad trade.
      This led to a broader conversation: Aaron's original plan was always
      for the *tool* to assemble a complete draft, then bring it to Claude
      directly (outside the tool) for full-manuscript editorial review -
      Claude's strength is long-document coherence; Grok has struggled with
      large files on this project. Separately, Sammy (Grok-based) was
      confirmed to require a PAID SuperGrok subscription specifically for
      the Custom Agent feature - free Grok only offers Custom Instructions,
      not persistent agents. Aaron wants to end the SuperGrok subscription
      once the tool is complete.
      **Final decision: split by job, not by subscription.**
      - **Sammy keeps doing generative/day-to-day story & canon work**
        (scene prompts, lore, "Update webtool" pushes) - whichever
        platform/tier that ends up living on is a separate decision from
        this one, not yet finalized.
      - **Claude does the heavy editorial pass on the FULL assembled
        manuscript**, once "Full draft assembly function" (see above) is
        built. This was Aaron's original plan and remains the plan -
        confirmed explicitly, not a fallback.
      - **DONE 2026-06-22 (follow-up):** the in-tool LanguageTool tool was
        NOT removed after all — Aaron asked to keep it, renamed away from
        "Coach" to **"Proofreader"** (an honest description of a mechanical
        pass, not implying contextual/editorial quality), with an added
        **"Apply All Suggestions"** button. Apply All uses a verified-safe
        right-to-left offset algorithm (sort issues by position descending,
        apply rightmost first) so applying multiple fixes in one click never
        corrupts other pending offsets — tested with 3 scattered issues in
        one paragraph, all applied correctly in a single click.
      - **DONE 2026-06-22:** "Send to Sammy" built and verified - copies the
        current scene's title, guidance summary, and live draft text to the
        clipboard in clean, ready-to-paste format, with "COPIED" button
        feedback. Confirmed via real clipboard read-back in testing.

- [ ] Main writing area layout/visual polish — intentionally deferred until
      after the fantasy-themed visual mockup is complete and broken down into
      skinnable elements (per Aaron, 2026-06-21).
- [ ] Open question: are the `safeGetItem`/`safeSetItem` localStorage wrapper
      functions (mentioned in the 2026-06-21 CHANGELOG entry) still needed?
      They do not currently exist in `writing.html`. Likely a holdover from
      a Grok-sandbox-specific issue that may not apply to Erica's actual
      browser environment — needs a decision, not urgent.
- [ ] **Bring desktop's Notepad up to date with mobile's new behavior.**
      Mobile-only fixes/changes landed 2026-08-05 (later round) that desktop
      never received, by design (mobile-port work is scoped mobile-only
      unless Aaron says otherwise): `hideNotepad()`'s save-before-close
      reordering (a save/cleanup error can no longer block the panel from
      closing), the derived note-list title (first 3 words + "…" instead of
      the literal "Untitled Note" when a note has no explicit title), and
      whatever the eventual fix turns out to be for the nav-button
      toggle-close bug (still being chased as of this entry — see
      CHANGELOG). Desktop's Notepad shares the same underlying functions for
      some of this (`saveCurrentNote`, `renderNotesList`) so it likely
      already inherits parts of it for free; needs a deliberate check once
      the mobile side is fully settled, not before.

## Questions Needing Erica's Input (Not Yet Reviewed)

See `Erica-Questions-Writing-Tool.md` for the full list compiled during dev
work. Many may already be answered by choices made since 2026-06-21
(per-scene drafts, status selector behavior, leaving-review confirmation) —
this needs a pass to mark which questions are now resolved vs. still open,
rather than assuming the whole list is still live.

---

*Last Updated: 2026-07-27*
