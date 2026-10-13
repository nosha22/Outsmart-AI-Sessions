# How to use the AI Sessions Board

**Prompt & Circumstance — AI Sessions Board** is a single-file web page for collecting what
people want to **share** and what they want to **learn** before an AI session, voting on the
learn requests, and turning the result into a draft agenda.

The whole app is one HTML file with inline CSS and JavaScript — no build step, no dependencies.

- `index.html` — the board
- `board.html` — byte-for-byte identical copy of `index.html`

Editing one and not the other will make them drift; keep changes in sync (or drop one).

## Running it

Open `index.html` in a browser. Double-clicking it works for everything **except** Google Sheet
sync: Google rejects requests from a `file://` page (origin `null`), so serve it over HTTP if you
want sync:

```sh
python3 -m http.server 8000     # then open http://localhost:8000/index.html
```

For a group, host it anywhere static (GitHub Pages, internal web server) and share the URL.

## Running a session

1. **Fill in the header** — `Session` (e.g. `15 Sep 2026`) and `You` (your name). The name in
   `You` is what gets attached to your votes, and it pre-fills the share form's author field.
2. **Everyone adds items.**
   - *I want to share* — title, one or two lines of detail, your name, and a format:
     Lightning (5 min), Demo (10 min), Deep dive (20 min), or Just a link (no talk).
   - *I want to learn* — the question, optional context, optional name (leave it blank to stay
     anonymous), and an urgency: Curious, Blocked on it, or Team-wide need.
3. **Vote** on learn requests by clicking the vote box. Clicking again removes your vote. Votes
   are recorded by the name in `You` (or `anon` if it's empty), so one person is one vote per
   item — and hovering the box shows who voted.
4. **Track progress** with each item's `done` button. Done items grey out, sort to the bottom,
   and get tagged `presented` (shares) or `covered` (learn requests). `undo` reverses it.
5. **Generate agenda** builds a timed 60-minute running order from the not-done items — welcome,
   up to 5 shares timed from their chosen format (`Just a link` items are skipped), then the
   top-voted learn requests. Copy the text into the meeting invite.

Learn requests sort by vote count, so the board is always showing you the agenda in priority
order before you even press the button.

## Where the data lives

By default, everything is in **that browser's localStorage only** (keys `ai-sessions-board-v1`,
`ai-sessions-api-url`, `ai-sessions-outbox`). Two people on the same page do *not* see each
other's items. A cleared browser profile, private window, or different machine means a fresh
board.

To move data around, use the toolbar:

| Button | What it does |
| --- | --- |
| **Export JSON** | Downloads the full board (items, votes, status) — the way to hand over to the next facilitator. |
| **Import JSON** | Loads a previously exported file. |
| **Export to Google Sheet** | Builds a CSV; copy to clipboard and paste into a Sheet, or download the file and use *File ▸ Import*. |
| **Import from Google Sheet** | Paste rows (or choose a CSV) from a Sheet or Google Form responses. The header row is auto-detected, existing items are kept, duplicates skipped. |
| **Clear board** | Wipes the local board. |

CSV columns, in order: `Type, Title, Details, Submitted by, Format / urgency, Votes, Voters,
Status, Session, Added`. Blank names stay blank, so anonymity survives the round trip.

## Optional: live shared board via Google Sheets

With an Apps Script Web App connected, every add, vote, and status change is POSTed to the
Sheet and everyone opening the board sees the same data.

1. Create a Google Sheet and open **Extensions ▸ Apps Script**.
2. Write a script with a `doPost(e)` that appends the posted JSON to the sheet and a `doGet(e)`
   that answers `?action=list`. The board expects JSON responses:
   - `doGet` → `{ ok: true, items: [ { id, kind, title, desc, who, extra, votes: [...], done, at } ] }`
     where `kind` is `"share"` or `"learn"`.
   - `doPost` → `{ ok: true }`, or `{ ok: false, error: "…" }` to reject a write.
   - Posted payloads are `{ action: "add" | "vote" | "done" | … }` plus the item fields
     (`vote` sends `{ id, voter }`, `done` sends `{ id, done }`).
3. **Deploy ▸ New deployment ▸ Web app**, with **Who has access: Anyone**, and copy the `/exec`
   URL.
4. Either paste that URL into the **⚙** panel on the board (per-browser, overrides the built-in
   one), or bake it in for everybody by editing the `SHEET_URL` constant near the top of the
   `<script>` block in `index.html`.

The ⚙ panel's buttons: **Connect** (save + immediately read the Sheet), **Test** (re-check
reachability and retry queued writes), **Disconnect** (back to local-only; items are kept),
**Use built-in** (discard your override and restore the URL baked into the file).

### Reading the status pill

| Pill | Meaning |
| --- | --- |
| `local only` | No Sheet connected — data stays in this browser. |
| `live` | Connected; writes are landing. |
| `saving (n)` | Writes in flight. |
| `retry (n)` / `error` | Last write failed — hover for the reason. |

Writes go through an outbox in localStorage (capped at 200), so a flaky connection retries
rather than losing changes; a `4xx` response is dropped instead of blocking everything behind
it. localStorage remains the local source of truth, so the board keeps working offline.

### If sync fails

- **URL contains `/a/macros/`** — it's restricted to your Workspace domain and redirects to a
  sign-in page. Redeploy with *Who has access: Anyone*; the working URL has no `/a/macros/`.
- **Must be the `/exec` URL**, not `/dev`.
- **Opened from disk (`file://`)** — Google rejects the `null` origin. Serve over HTTP.
- **"Failed to fetch"** — usually one of the above, or you're offline.

Note: `index.html` currently ships with a `/a/macros/` Workspace-restricted `SHEET_URL` baked
in, which will not work outside that domain. Replace it with your own `/exec` URL, or clear it
to make the file local-only by default.

## Known gaps

The in-page help text points at `apps-script/SETUP.md`, `GO-LIVE.md`, `test/serve.js`, and
`test/mock-sheet.js`. None of those exist in this repo — the Apps Script sketch above is the
substitute until they're added.
