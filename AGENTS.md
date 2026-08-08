# AGENTS.md — Fast Orientation for Any Agent Touching This Project

Read this before reading anything else. Five minutes here saves hours.
Full detail lives in `Project_Governance_and_Protocols.md` — this is just
the load-bearing facts that are easy to get wrong. Sammy-specific
day-to-day rules are also extracted in `Sammy_Guardrails.md` and
`Sammy_Update_Webtool_Protocol.md` (they defer to that Governance
document).

## The one-sentence model
Sammy writes story canon to GitHub JSON. The webtool (`writing.html`) reads
it, layers the user's own status/draft decisions on top in localStorage, and
never writes back to Sammy's file. Nobody else writes to GitHub for this
project except Sammy, and only on an explicit "Update webtool" command.

## Facts that took real back-and-forth to establish — don't re-derive them
- **Grok "Projects" are not used for this work, deliberately.** Projects run
  in a sandbox that can silently drift from the real repo/Drive files —
  a sandboxed Dom instance could edit its own mirror, report success, and
  never actually sync back out. This is believed to be the root mechanism
  behind the worst corruption incident (see `workflow-memory.md`). Aaron
  decided (2026-06-22) to avoid Projects entirely and rely on the external
  refresh protocol instead (GitHub repo + Drive docs + "Refresh yourself"
  trigger) — every agent reads/writes the same real files, no sandboxed
  mirror. If a future agent suggests reviving Projects for context
  management, point back to this note first.
- **Sammy decides if a change needs review, not the tool.** The tool reads
  `scene.status` + `scene.needsReviewReason` as Sammy's editorial call. It
  does NOT diff text itself to decide this. (Earlier code tried to; it was
  wrong and was removed 2026-06-22.)
- **Review status: tool sets it, only the user clears it.** Never offer
  "Review" as a pickable dropdown option — it only appears when it's the
  synced state. Clearing it requires a confirm dialog.
- **The tool persists user status/draft choices locally, separate from
  Sammy's JSON.** A sync must never silently revert a user's "Complete" back
  to whatever Sammy's file currently says, unless Sammy explicitly re-flagged
  that scene for review in this sync.
- **Drafts are per-scene, not global.** A single shared `current-draft` key
  was a real bug (fixed 2026-06-22) — verify any new draft-related feature
  is scoped by scene ID.
- **Google Drive may contain stale, pre-migration files.** Specifically, an
  early `Big_Tiff_Scene_Structure.json` copy in Drive uses an old single-
  `prompt`-field schema. The only live source of truth for scene structure
  is the GitHub repo, using the 7-part field structure
  (`Big_Tiff_Standardized_Scene_Prompt_Template.md` v1.3+).
- **"Done" means tested in a real browser, not "code was written."** The
  project's worst incident (see `workflow-memory.md`) was an agent
  confidently reporting verified work that had actually reverted. If you
  have headless-browser tooling available, use it before claiming something
  works — reading the code is not verification.
- **This is a gift for Erica, who has ADD.** Every design choice should
  reduce decisions and friction for her, not add steps. When in doubt
  between a more "correct" flow and a calmer one, prefer calmer.
- **Mobile UI work has a fast-iteration loop that stops short of the real
  site, on purpose.** Commit + push to `claude/mobile-port` only; that
  branch's GitHub Pages preview auto-rebuilds in ~1 min. **Never** promote
  to `bigtiffsworld.com` / the `big_tiff_launchpage` repo as part of that
  loop — promotion is a separate, explicit step Aaron requests only after
  approving changes on the preview URL. Full detail, current deployment
  status, and mobile architecture notes: `Mobile_Port_Handoff.md`.
- **The achievements/usage-tracking engine (188 achievements, encrypted
  `progress.json`) shipped 2026-07-26 — UI (icons, achievement book,
  trophy shelves) has not.** Don't assume the engine needs building if a
  request sounds like it's asking for achievement logic; check
  `CHANGELOG.md`'s 2026-07-26 entries and `getAchievementBook()` first.

## Where things actually live
| What | Where | Who writes it |
|---|---|---|
| `writing.html` (the tool) | GitHub: `footeprint-prog/big-tiff-storyforge` | Dom / whoever is doing tool dev |
| `Big_Tiff_Scene_Structure.json` | same GitHub repo | Sammy only, on "Update webtool" |
| `Big_Tiff_Library_Entries.json` | same GitHub repo | Sammy only |
| Master Story Bible + lore docs | Google Drive, Big Tiff folder | Sammy |
| `CHECKLIST.md` / `CHANGELOG.md` | same GitHub repo | whoever does tool dev, after real verification |

## If you're about to say something is fixed/working/done
Stop and check: did you actually run it, or did you just read the code and
reason about what it should do? If the latter, say so explicitly rather than
reporting it as verified.

## Token efficiency — this file is a single ~600KB HTML file, act like it
`writing.html` is one large file with a lot of unrelated history in it
(CSS, JS, and markup for both desktop and mobile interaction models, plus
years of dated inline comments). Full-file reads and broad exploration
are expensive here specifically. Concrete habits that have actually saved
real time on this project, not generic advice:
- **Grep for the specific function/id/class before reading anything.**
  `grep -n "functionName"` to find the line, then `Read` with an
  `offset`/`limit` around it - almost never need the whole file. The
  mobile-vs-desktop split means a feature's mobile CSS, desktop CSS, and
  shared JS are usually far apart in the file; grep finds all three
  faster than scrolling.
- **Check `isMobileMode()` gating before assuming something needs
  "porting" between desktop and mobile.** A large fraction of what looks
  like a cross-platform gap turns out to be one shared, ungated function
  - desktop already has it for free. Confirmed repeatedly (2026-08-04
  desktop parity pass): most of a 30+-item comparison needed zero code
  changes once actually checked against the code instead of assumed from
  changelog prose.
- **When a report can't be reproduced after 2-3 real attempts, say so and
  ask for repro details instead of testing every adjacent feature "just
  in case."** Broadening test surface indefinitely without new evidence
  burns tokens without converging - a request for repro steps (browser,
  logged in/guest, exact symptom) is more efficient than a fourth round
  of the same testing that already found nothing.
- **Test a private/incognito tab before suspecting the code**, whenever a
  report is "works elsewhere but not in this one browser/tab, even after
  reload." Isolates stale local storage from a real bug in about ten
  seconds - the 2026-08-03 "Safari bug" cost a full round of code-level
  investigation (including a revert-then-re-add) before this ten-second
  check would have shown it wasn't code at all.
- **Delegate large, mostly-mechanical reading to a subagent and keep only
  the synthesized result.** Reading a long CHANGELOG history to extract a
  structured list is exactly this shape - hand the raw reading to an
  agent, do the judgment/synthesis yourself from its output, don't also
  re-read the source in the main thread.
- **Don't re-verify functionality a previous session already tested and
  documented as working**, absent a new, specific complaint. Trust
  CHANGELOG "Verification notes" sections rather than re-deriving them -
  they exist precisely so the next session doesn't have to.
- **Match doc-update verbosity to what's actually new.** Append a short
  `**UPDATE <date>:**` note to an existing CHECKLIST/CHANGELOG entry
  rather than rewriting it; point to where fuller detail already lives
  instead of repeating it.
