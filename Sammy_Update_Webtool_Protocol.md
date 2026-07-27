# Sammy — "Update Webtool" Protocol

> This document extracts and clarifies the exact process Sammy must follow
> when given the **"Update webtool"** command, for day-to-day use by Grok,
> Aaron, Erica, and Sammy. It introduces no new steps and softens none —
> everything below is drawn directly from `Project_Governance_and_Protocols.md`
> and `AGENTS.md`. **If anything here conflicts with those two documents,
> those two documents take precedence.** Flag any conflict you notice
> rather than resolving it from this file alone.
>
> The general, always-on rules Sammy operates under (ownership, draft
> protection, what Sammy must never do) live in `Sammy_Guardrails.md` and
> apply throughout everything below — they are not repeated here.

---

## When this applies

Whenever Aaron or Erica gives the command **"Update webtool"**, or any
instruction that requires updating `Big_Tiff_Scene_Structure.json` or
`Big_Tiff_Library_Entries.json` in GitHub.

---

## Step 0 — Pre-Update Requirements (mandatory, before touching anything)

- Confirm you are working from the **latest version** of all relevant
  files.
- Read the current `Big_Tiff_Standardized_Scene_Prompt_Template.md`
  (7-part structure) before creating or updating any scene prompts.
- Verify current canon by referencing the Master Story Bible.
- **If unsure about any canon detail — stop and ask** Aaron or Erica
  before proceeding. Do not guess.

## Steps 1–6 — The process

1. **Apply the 7-part Standardized Scene Prompt Template** when creating
   new scene prompts or updating existing ones.
2. **Update `Big_Tiff_Scene_Structure.json`** in GitHub with those
   changes.
3. **Set any new or significantly changed scenes to `status: "review"`**
   and populate `needsReviewReason` with a clear explanation of what
   changed and why.
4. **Review and update relevant Library entries** in
   `Big_Tiff_Library_Entries.json` in the same pass — especially where
   scene work or canon changes affect Library content.
5. **Commit**, using the required commit message format (below).
6. **Provide a short summary** of what changed in both the Outline and
   the Library, and why.

---

## Commit Message Standards

Every commit Sammy makes must follow this format:

```
[Type] Brief description of change

- What was changed
- Why it was changed
- Reference to related canon or template version if relevant
```

---

## Reminder

`Sammy_Guardrails.md`'s "What Sammy Must NOT Do," Error Handling, and
Human Oversight sections apply throughout this entire process. If
anything in Step 0–6 above ever seems to require doing one of those
prohibited things, stop and ask — don't resolve it by proceeding anyway.
