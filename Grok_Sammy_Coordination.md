# Grok ↔ Sammy Coordination

> This document extracts and clarifies how Grok and Sammy are expected to
> interact, for day-to-day use by Grok, Aaron, Erica, and Sammy. It
> introduces no new rules and softens none — everything below is drawn
> directly from `Project_Governance_and_Protocols.md` and `AGENTS.md`,
> except the one item explicitly marked as a flagged observation rather
> than an extracted rule. **If anything here conflicts with those two
> documents, those two documents take precedence.** Flag any conflict you
> notice rather than resolving it from this file alone.

Kept light, per Aaron's request — this is a short coordination note, not
a third governance document.

---

## Roles, as documented in `Project_Governance_and_Protocols.md`

- **Grok** — Project Orchestrator & Chief Architect: vision, architecture,
  and cross-agent alignment. Steps in for strategic or high-level
  decisions.
- **Sammy** — Story & Canon Specialist: story development, scene prompts,
  and Library entries (see `Sammy_Guardrails.md` for the full rule set).

## Two Grok instances, one set of rules

Aaron and Erica each run their own Grok instance, and each may be running
a version of Sammy. **Both versions of Sammy must follow the identical
core rules** in `Sammy_Guardrails.md`, even where tone and detail level
differ (Erica's is deliberately calmer and phone-friendly).

## Alignment mechanism

- Anyone can say **"Refresh yourself"** in either Grok instance to trigger
  a check that both sides still agree on current protocols and file
  locations.
- When a significant change is made to architecture, protocols, or file
  locations, it should be reflected in **both** instances within a
  reasonable time.
- Grok is responsible for keeping the governance documents and Quick
  Reference up to date, and for keeping both instances aligned.

## Where Claude / Claude Code fits

Claude Code builds and maintains `writing.html` directly via GitHub —
this replaces the "Webcraft Master Dom" persona described in
`Project_Governance_and_Protocols.md`. Claude does **not** touch
`Big_Tiff_Scene_Structure.json` or `Big_Tiff_Library_Entries.json` — those
remain exclusively Sammy's, written only via "Update webtool."

## Where ChatGPT fits

Not described in either source document. Rather than guess at a role,
this is left as an open item — flag it to Aaron for a decision if it
becomes relevant, don't assume a scope for it here.

---

## ⚠ Flagged observation, not resolved here

`Project_Governance_and_Protocols.md` currently carries a banner (added
2026-07-26) describing "the multi-agent structure below (Grok as
Orchestrator, 'Webcraft Master Dom' building the tool)" as historical, in
one breath. But Aaron's direction (2026-07-27) is that **Grok's
coordinating/orchestrator role between Sammy, Aaron, Erica, Claude, and
potentially ChatGPT is still very much active** — only the Dom persona
(tool-building) has actually been replaced, by Claude Code.

This document treats Grok's orchestrator role as current, per that
direction. The banner's wording hasn't been corrected to draw that
distinction yet. That's a separate, small follow-up edit to an existing
file — not made here, and not made without explicit approval, per the
brief this document was written under.
