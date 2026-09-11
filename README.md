# aman-homepage

Aman's personal homepage — a private, email-gated single page with a morning
briefing (date, META quote, headlines) and his project map.

## How it's protected

- The page sits behind a Supabase magic-link gate (`index.html`): only
  `amaan.gupta19@gmail.com` can request a sign-in link, and public sign-ups
  are disabled in the Supabase project.
- Project names/statuses live in the Supabase `projects` table (RLS: only
  authenticated reads), so they never sit in this public repo.
- `data.json` holds only public-ish briefing data (date, META quote,
  headlines) and is refreshed daily by the `daily-refresh` Action.

## Project timeline (time machine)

- `project_history` is an append-only event log: every project creation,
  status/layer/rename change, removal, and timestamped note lands here with
  `occurred_at`. RLS: authenticated reads only.
- Event conventions (the homepage scrubber replays these oldest-first):
  `added` (new_value=status, note='layer:<Layer>'), `status`
  (old_value→new_value), `layer`, `renamed`, `removed`, `note` (no state
  change), `baseline` (like added, unknown creation time).
- The schema + full backfill live in `supabase/project_history.sql`.
  Backfilled from the project-map artifact: real creation timestamps for all
  20 projects + the 3 timestamped project updates.
- The Project Map tab has a "Time machine" scrubber (date picker, ◀/▶ day
  step, Live button) that reconstructs the board as of any past date and
  shows that day's event feed. Dates are America/New_York.

## Keeping Supabase in sync (Scout's standing rule)

Supabase (`projects` + `project_history`) is the live source the homepage
reads — it gets the SAME updates as the project-map artifact. Whenever a
project is added/changed:
1. Upsert `projects` (name unique; set layer + status).
2. Append the matching event(s) to `project_history` with the real
   timestamp (use `added`/`status`/`layer`/`renamed`/`removed`/`note`).
Writes go through the Supabase dashboard SQL editor (logged-in session) —
the anon key is read-only by RLS design, and no service keys are stored
anywhere.

## Files

- `index.html` — the page (gate + briefing + project map).
- `data.json` — refreshed daily by the Action below.
- `.github/workflows/refresh.yml` — daily 7am ET refresh. The Python
  refresher is embedded in the workflow (heredoc); the canonical copy for
  local testing lives in `scripts/refresh.py` (not deployed).

## Supabase setup (already done, recorded here)

- Project `aman-homepage`, region us-east-1, free plan.
- Table `projects(name text unique, layer text, status text)` with RLS
  policy allowing `select` to the `authenticated` role only.
- Auth: Email provider on, "Allow new users to sign up" OFF, owner invited.
- URL Configuration must allowlist the Pages URL
  (`https://amanguptaBay.github.io/aman-homepage/`) as a redirect URL,
  otherwise magic-link sign-in can't return to the site.
