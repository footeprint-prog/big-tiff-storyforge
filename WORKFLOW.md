# WORKFLOW.md — Big Tiff StoryForge Writing Tool

**Purpose**  
This document defines the required workflow for any agent (Grok or future agents) when making changes to `writing.html` and related files. It emphasizes safe Git-based version control and backup.

**Scope**  
Applies to all edits involving `writing.html`, `workflow-memory.md`, and supporting documentation in the `big-tiff-storyforge` repository.

**Last Updated:** 2026-06-21  
**Maintained by:** Project Dom Architect

---

## 1. Core Principles (Non-Negotiable)

- **Local file is the Source of Truth.** GitHub serves only as backup and version history.
- **Small, incremental changes only.** Large structural replacements are forbidden.
- **Verify functionality before committing.** Always test in-browser behavior and check the console after edits.
- **Always read real file content** before any push operation.
- **Zero placeholder usage.** Never pass placeholder strings when pushing files.
- **Mandatory self-review** before considering any change complete.

---

## 2. Recommended Local Development Workflow

```bash
# Clone or update the repository
git clone https://github.com/footeprint-prog/big-tiff-storyforge.git
cd big-tiff-storyforge

# Create a dedicated feature branch
git checkout -b feature/writing-tool-<short-description>

# Make small, precise edits to writing.html
# Prefer targeted replacements over large block changes

# After significant changes:
# 1. Open writing.html in a browser
# 2. Test key flows (scene selection, editor, panels, canon sync, notepad)
# 3. Check browser console for errors

# Commit locally with clear messages
git add writing.html
git commit -m "feat: connect scene selection to editor and load draft (Step 4)"
```

---

## 3. GitHub Push / Backup Workflow

### Primary Recommended Method: Standard Git

```bash
git push origin feature/writing-tool-<name>
```

Create a Pull Request for review when appropriate.

### Alternative Method: `github___push_files` Tool (Use Cautiously)

This tool can be used but requires strict adherence to the following rules due to past reliability issues:

**Rules for Using `github___push_files`:**

1. **Always read the actual file content first**:
   ```python
   with open("writing.html", "r", encoding="utf-8") as f:
       actual_content = f.read()
   ```

2. Pass only `actual_content` — never placeholder strings.

3. **Immediately verify** the push by checking the raw URL:
   ```
   https://raw.githubusercontent.com/footeprint-prog/big-tiff-storyforge/main/writing.html
   ```

4. If the raw file does not contain the expected real content, consider the push failed and investigate before continuing.

**Recommendation:** Prefer standard `git push` for most work. Use `github___push_files` only when necessary, and always verify afterward.

---

## 4. Verification Checklist (Before Every Push)

- [ ] Change is small and focused
- [ ] HTML is structurally valid
- [ ] JavaScript executes without console errors
- [ ] Key interactions tested in browser
- [ ] Real file content was read before pushing
- [ ] Commit message is clear and follows format: `type: description (Step X)`
- [ ] Local commit created before pushing to GitHub

---

## 5. Branching & Commit Guidelines

| Branch                  | Purpose                              | Usage                              |
|-------------------------|--------------------------------------|------------------------------------|
| `main`                  | Stable, verified version             | Protected – merge only after review |
| `feature/writing-tool-*` | Active development                   | Normal day-to-day work             |
| `hotfix/*`              | Urgent fixes                         | Rare                               |

**Commit Message Format Example:**
- `feat: add per-scene draft persistence`
- `fix: restore missing status selector in scene view`
- `docs: update WORKFLOW.md with push verification rules`

---

## 6. Rollback Procedures

```bash
# View recent changes to a file
git log --oneline --follow writing.html

# Revert a single file to a previous state
git checkout <commit-hash> -- writing.html

# Full branch reset (use with extreme caution)
git reset --hard <commit-hash>
```

After any rollback, re-test functionality before pushing.

---

## 7. Agent Self-Review Requirements

Before marking work as complete, the agent must:

- Perform a structured self-review of accuracy, side effects, and test coverage.
- Confirm adherence to the "small increments" rule.
- Verify both HTML structure and JavaScript runtime behavior.
- Update `workflow-memory.md` if new failure patterns or improvements were discovered.

---

## 8. Relationship to Other Documents

- `workflow-memory.md` — Internal incident log and lessons learned (historical context).
- `WORKFLOW.md` — This document. The active, prescriptive guide agents should follow.

When in doubt, follow this document. Update it when better processes are proven.

---

**End of Document**