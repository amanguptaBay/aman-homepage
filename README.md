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
