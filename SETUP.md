# GIDF ABC Claim Board — Setup Guide

One-time setup to get the HTML running as a live, shared team claim board at a public URL.

**Total time:** ~15 minutes. Only Swapnil needs to do this once.

**What you'll need:**
- GitHub account (free)
- GitHub Desktop app installed on your Mac
- Supabase account (free, sign in with GitHub)

**The 7 steps at a glance:** create Supabase project → run SQL → grab keys → paste into `index.html` → create GitHub repo + clone → double-click sync script → enable GitHub Pages.

---

## Step 1 — Create the Supabase project (5 min)

1. Go to **https://supabase.com** → **Sign in with GitHub**.
2. Click **New project**.
   - Name: `gidf-claims` (anything is fine)
   - Database password: generate one and save it somewhere (you won't need it day-to-day)
   - Region: pick the one closest to Belgium (e.g., `eu-central-1` / Frankfurt)
3. Wait ~2 minutes while it provisions.

## Step 2 — Create the tables (2 min)

In the Supabase dashboard, click the **SQL Editor** icon in the left sidebar → **New query** → paste the block below → **Run**.

```sql
-- Claims table: who's taking which task
create table gidf_claims (
  task_id text primary key,
  primary_name text,
  extra_name text,
  updated_at timestamptz default now()
);

alter table gidf_claims enable row level security;
create policy "Anyone can read claims"   on gidf_claims for select using (true);
create policy "Anyone can insert claims" on gidf_claims for insert with check (true);
create policy "Anyone can update claims" on gidf_claims for update using (true) with check (true);
create policy "Anyone can delete claims" on gidf_claims for delete using (true);
alter publication supabase_realtime add table gidf_claims;

-- Task edits table: renamed task text + admin notes, keyed by the hash of the ORIGINAL text
create table gidf_task_edits (
  task_id text primary key,
  edited_text text,
  edited_detail text,
  note text,
  updated_at timestamptz default now()
);

alter table gidf_task_edits enable row level security;
create policy "Anyone can read edits"   on gidf_task_edits for select using (true);
create policy "Anyone can insert edits" on gidf_task_edits for insert with check (true);
create policy "Anyone can update edits" on gidf_task_edits for update using (true) with check (true);
create policy "Anyone can delete edits" on gidf_task_edits for delete using (true);
alter publication supabase_realtime add table gidf_task_edits;

-- Custom tasks table: admin-added tasks (beyond the ones baked into the HTML)
create extension if not exists "pgcrypto";
create table gidf_custom_tasks (
  id uuid primary key default gen_random_uuid(),
  section_id text not null,
  task_text text not null,
  task_detail text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

alter table gidf_custom_tasks enable row level security;
create policy "Anyone can read custom"   on gidf_custom_tasks for select using (true);
create policy "Anyone can insert custom" on gidf_custom_tasks for insert with check (true);
create policy "Anyone can update custom" on gidf_custom_tasks for update using (true) with check (true);
create policy "Anyone can delete custom" on gidf_custom_tasks for delete using (true);
alter publication supabase_realtime add table gidf_custom_tasks;
```

This creates two tables — one for claims, one for task-text edits — both readable/writable by anyone with the anon key. The HTML gates edit mode behind a code separately.

## Step 3 — Grab your project URL + anon key (1 min)

In the Supabase dashboard: **Project Settings** (gear icon) → **API**.

Copy two values:
- **Project URL** (looks like `https://xxxxxxxxxxxxxxxx.supabase.co`)
- **Project API key → `anon` `public`** (a long string starting with `eyJ...`)

## Step 4 — Paste them into index.html + set the edit code (2 min)

Open `index.html` in a text editor. Near the bottom, find this block:

```js
const SUPABASE_URL = '__PASTE_YOUR_SUPABASE_URL_HERE__';
const SUPABASE_ANON_KEY = '__PASTE_YOUR_SUPABASE_ANON_KEY_HERE__';

// Edit code — share with trusted people who can edit task text.
const EDIT_CODE = 'gidf-2026';
```

Replace the two `__PASTE__` placeholders with your Supabase values.

**Set your edit code** — change `'gidf-2026'` to whatever you want. Anyone with this code can rename tasks. Share it in the ABC WhatsApp group to the people you trust to edit the list. If the code leaks, just change it here and redeploy.

Save.

## Step 5 — Create the GitHub repo (2 min)

1. Go to **https://github.com/new**
2. Repo name: `gidf-abc-claims` (or whatever you like)
3. Visibility: **Public** (GitHub Pages requires this on free accounts)
4. Leave "Add a README" **unchecked** (we have our own)
5. Click **Create repository**

On the next screen you'll see the repo URL — copy it.

Open **GitHub Desktop** → **File → Clone repository** → paste the URL → choose where to save the local folder → **Clone**. This downloads an empty copy to your Mac.

## Step 6 — Sync files to GitHub (1 min)

In the `github-pages/` folder (this folder), **double-click `sync-to-github.command`**.

- **First run** — it asks for the path to your local clone. In GitHub Desktop, right-click the repo → **Show in Finder**, then drag that folder into the Terminal window. Press Enter.
- The script rsyncs `index.html`, `SETUP.md`, and `README.md` into the root of your cloned repo, then opens GitHub Desktop.
- In GitHub Desktop: add a commit message → **Commit to main** → **Push origin**.

Every future update, just double-click `sync-to-github.command` again — it remembers the target path.

> **Note:** `index.html` lands in the **root** of the repo, not inside a subfolder. GitHub Pages serves the root as your site's homepage.

## Step 7 — Enable GitHub Pages (1 min)

On the repo page on GitHub:
1. **Settings** → **Pages** (left sidebar)
2. **Source:** Deploy from a branch
3. **Branch:** `main`, folder: `/ (root)`
4. **Save**

After ~1 minute, your site is live at:
```
https://YOUR_USERNAME.github.io/gidf-abc-claims/
```

Share that URL in the ABC WhatsApp group. Done.

---

## How to verify it works

1. Open the URL in two different browsers (e.g., Chrome + your phone).
2. In one, claim a task by picking a name from the dropdown.
3. Within ~1 second, the claim appears in the other browser.
4. The green dot in the bottom-right says **Live — synced with team**.

If the dot is orange (`Local-only`) you haven't filled in the Supabase keys. If red (`Offline`), check the keys are correct.

## How the edit-code works

Anyone opening the URL can **claim** tasks. Only people who know the **edit code** can **rename** task text or add **admin notes**.

- Click the **✒ Edit** button (bottom-left corner)
- Enter the code
- Click any task text to rename it
- Click the "add a note…" field below any task to pin context for the team (e.g. "caterer confirmed — deposit paid Apr 24")
- Click **+ Add task** at the bottom of any subsection to create a new task — name, optional detail, and it appears for everyone immediately
- Click the small × on the right of a custom task to delete it
- Click away to save; changes sync to everyone within ~1 sec

Edits sync live to everyone via Supabase. Renames, details, and notes are stored in `gidf_task_edits`, keyed by the hash of the original task text — rename safely without breaking existing claims.

**To revert an edit** to the original text: in edit mode, clear the field → click away → it falls back to the original from the HTML source.
**To remove a note:** clear the text and click away.

---

## Security note

The `anon` key is safe to commit publicly — it only has the permissions you gave it via Row Level Security. Anyone with the URL can claim/unclaim tasks, which is the point.

**About the edit code:** it lives in plain text in `index.html`, so technically anyone who opens devtools → view source can read it. For a trusted ABC team of 13, this is fine — the code is mostly a guardrail against accidental edits and random visitors. If you want tighter security, ask me to switch to a hashed code (~2 minutes of work).

---

## Troubleshooting

**"Local-only (not configured)"** — SUPABASE_URL or SUPABASE_ANON_KEY still has the placeholder text. Replace both.

**"Offline — claims saved locally"** — The keys are wrong, or the `gidf_claims` table wasn't created. Re-check Step 2.

**Claims aren't syncing to other browsers in real-time** — The realtime publication step at the end of the SQL may have been skipped. Run just this line:
```sql
alter publication supabase_realtime add table gidf_claims;
```

**Someone wants to remove a claim** — Set the dropdown back to `— pick your name —` and clear the teammate box.

**Edited text didn't save** — Make sure the `gidf_task_edits` table exists (re-run Step 2 SQL if unsure). Also check the sync-status pill is green when you exit edit mode.

**Want to reset ALL edits** — In Supabase SQL editor: `delete from gidf_task_edits;` reverts every task to its original HTML text.

**Already deployed before notes were added?** Run this one-line migration in Supabase SQL editor to add the `note` column:
```sql
alter table gidf_task_edits add column if not exists note text;
```

**Already deployed before custom tasks were added?** Run this migration to add the `gidf_custom_tasks` table:
```sql
create extension if not exists "pgcrypto";
create table gidf_custom_tasks (
  id uuid primary key default gen_random_uuid(),
  section_id text not null,
  task_text text not null,
  task_detail text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);
alter table gidf_custom_tasks enable row level security;
create policy "Anyone can read custom"   on gidf_custom_tasks for select using (true);
create policy "Anyone can insert custom" on gidf_custom_tasks for insert with check (true);
create policy "Anyone can update custom" on gidf_custom_tasks for update using (true) with check (true);
create policy "Anyone can delete custom" on gidf_custom_tasks for delete using (true);
alter publication supabase_realtime add table gidf_custom_tasks;
```
