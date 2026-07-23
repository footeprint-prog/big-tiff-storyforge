# Big Tiff StoryForge — One-Time Setup

The tool runs entirely in the browser and stores its shared data as JSON in the
**public `big-tiff-data`** repo. No server, no Cloudflare. This guide covers the
three one-time setup steps. Do them in order.

> Security reality check: login here is a **soft gate** (keeps casual visitors
> out, not real security), and draft text in `big-tiff-data` is encrypted but
> only **"lightly private"** (a determined person could extract the key from the
> tool's source). Don't put anything truly sensitive in here beyond the novel.

---

## 1. Create the accounts (passwords)

Passwords are never typed into `accounts.json` directly — you generate a salted
**hash** for each one.

1. Open **`account-hash-generator.html`** (in this repo) in any browser. It runs
   locally; nothing is sent anywhere.
2. For each person, fill in username (lowercase, no spaces), role, password, and
   whether they can Sync Canon, then click **Add / update user**. Suggested:
   - `ericap` — role **primary** (full read/write).
   - `aaronf` — role **viewer**, Can Sync Canon = **yes** (the sync exception).
   - any other viewers — role **viewer**, Can Sync Canon = **no**.
3. Click **Copy** to copy the generated `accounts.json` block.
4. In the **`big-tiff-data`** repo, open `accounts.json`, replace the whole file
   with what you copied, and commit.

To change a password later, re-add that username in the generator and re-commit.

---

## 2. Turn on cloud saving for Erica (write token)

Only Erica's browser needs this — it's what lets her writing save to the cloud.
It's also offered automatically the first time she logs in as `ericap`. She can
re-open it anytime from the account menu → **Cloud saving…**.

1. Go to <https://github.com/settings/personal-access-tokens/new> (signed in as
   the account that owns `big-tiff-data`).
2. **Token name:** anything (e.g. "Big Tiff writing").
3. **Expiration:** longest available, or "No expiration".
4. **Repository access:** *Only select repositories* → **big-tiff-data**.
5. **Permissions:** *Repository permissions* → **Contents** → **Read and write**.
6. **Generate token**, copy it, and paste it into the tool's "Turn on cloud
   saving" box → **Save & verify**.

The token is stored **only in that browser** (localStorage) and is never
committed. If it ever expires, just repeat this and paste a fresh one. Without a
token the tool still works and saves locally on that device — it just won't sync.

Viewers do **not** need a token (they don't write). This is also the real
enforcement of read-only: no token = no ability to change shared data.

---

## 3. Serve the tool from the launch site (same origin)

So the launch-page login and the tool share a session, the tool is served from
the same site as `bigtiffsworld.com`. The copy of `writing.html` that goes live
lives in the **`big_tiff_launchpage`** repo at `app/index.html`.

**Promoting an update** (whenever `writing.html` changes here and is verified):
1. Copy `big-tiff-storyforge/writing.html` → `big_tiff_launchpage/app/index.html`.
2. Copy the `assets/` GIFs too if they changed.
3. Commit + push `big_tiff_launchpage` → GitHub Pages redeploys automatically.

`big-tiff-storyforge` stays the development source of truth; the copy step is the
single, deliberate "dev → live" moment.

---

## How it all fits together

- **Login** (launch page or tool) checks the entered password against the hashes
  in `accounts.json` — all in the browser.
- **On load**, the tool auto-syncs: it reads `drafts.json` / `checkpoints.json` /
  `stats.json` from `big-tiff-data` so every profile sees the same state, right
  where Erica left off. No manual "Sync" needed for that.
- **As Erica writes**, her browser encrypts each draft and commits it back to
  `big-tiff-data` using her token. If the same scene was changed on another
  device, the tool shows a **compare-and-choose banner** instead of overwriting —
  and the version she doesn't keep is saved as a Draft Pad checkpoint first, so
  nothing is ever lost.
- **Viewers** see everything read-only, can jot Notepad notes (not delete them),
  and — except `aaronf` — can't Sync Canon. They hold no token, so they can't
  change shared data even if they poke at the page.
