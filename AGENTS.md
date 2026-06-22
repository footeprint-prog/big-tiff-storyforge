# AGENTS.md — Fast Orientation for Any Agent Touching This Project

Read this before reading anything else. Five minutes here saves hours.
Full detail lives in `Project_Governance_and_Protocols.md` — this is just
the load-bearing facts that are easy to get wrong.

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
