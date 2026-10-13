# Prompt & Circumstance — AI Sessions Board

A single-file web page for planning AI sessions: people post what they want to **share** and what
they want to **learn**, vote on the learn requests, and the board turns the result into a timed
draft agenda.

No build step, no dependencies — one HTML file with inline CSS and JavaScript.

## Quick start

Open `index.html` in a browser. That's enough for everything except Google Sheet sync, which
needs a real HTTP origin:

```sh
python3 -m http.server 8000     # then open http://localhost:8000/index.html
```

For a group, host it on any static host (GitHub Pages, internal server) and share the URL.

## What it does

- **Share / learn columns** — shares carry a format (Lightning, Demo, Deep dive, Just a link);
  learn requests carry an urgency (Curious, Blocked on it, Team-wide need) and can be anonymous.
- **Voting** — one vote per person per learn request, keyed off the name in the `You` field;
  requests sort by vote count, so the board always shows priority order.
- **Done tracking** — items grey out, sort to the bottom, and get tagged `presented` / `covered`.
- **Generate agenda** — builds a 60-minute running order from the not-done items.
- **Import / export** — full-board JSON, plus CSV round-tripping with Google Sheets.
- **Optional live sync** — point the board at a Google Apps Script Web App and everyone sees the
  same data; writes queue in an outbox and retry, so it keeps working offline.

By default all data lives in that browser's localStorage only — two people on the same page do
not see each other's items until a Sheet is connected.

## Files

| File | |
| --- | --- |
| `index.html` | The board. |
| `board.html` | Byte-for-byte identical copy of `index.html` — keep changes in sync, or drop one. |
| `HOW-TO.md` | Full guide: running a session, data/export details, Sheet sync setup, troubleshooting. |

## Notes

- `index.html` ships with a Workspace-restricted (`/a/macros/`) `SHEET_URL` baked in, which will
  not work outside that domain. Replace it with your own `/exec` URL or clear it.
- The in-page help text references `apps-script/SETUP.md`, `GO-LIVE.md`, and files under `test/`
  that don't exist in this repo; `HOW-TO.md` covers that ground instead.

See [HOW-TO.md](HOW-TO.md) for the details.
