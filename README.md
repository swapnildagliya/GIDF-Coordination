# GIDF 2026 — ABC Claim Board

Live, shared task board for Gent India Dans Festival 2026 (May 1–3).

ABC dancers self-claim tasks from the checklist. Claims sync across everyone in real-time. Anyone with the edit PIN can rename task text.

## Stack

- **Hosting:** GitHub Pages (static)
- **Backend:** Supabase (Postgres + Realtime)
- **No framework** — vanilla HTML + JS

## Deploy

See [SETUP.md](./SETUP.md) for one-time setup (~15 min).

## Update workflow

1. Edit `index.html` in this folder (locally)
2. Double-click `sync-to-github.command` — it copies the files into your local clone and opens GitHub Desktop
3. Commit + push in GitHub Desktop
4. GitHub Pages redeploys in ~1 min

## Files

- `index.html` — the whole app (single file)
- `SETUP.md` — deployment guide
- `README.md` — this file
- `sync-to-github.command` — double-click to push updates (macOS)
