# TaraGourmet

A mobile-first recipe app with guest-scaled quantities, a guided/timed cooking
mode, and per-account accounts backed by Supabase.

## What's here

- `index.html` — the entire app (auth screen + Screens 1–4, styling, and logic)
- `schema.sql` — the Supabase table and Row Level Security policies to run once
- `manifest.json` — PWA manifest so it can be added to an iOS/Android home screen
- `sw.js` — service worker for basic offline caching

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) → New project
2. Once it's provisioned, open **Project Settings → API** and copy:
   - **Project URL**
   - **anon / public** key (not the `service_role` key — that one never
     belongs in client-side code)
3. Open **SQL Editor → New query**, paste in the contents of `schema.sql`,
   and run it. This creates the `recipes` table and locks it down with Row
   Level Security so each account can only see and edit its own rows.
4. In **Authentication → Providers**, email/password sign-up is on by
   default — nothing else to configure to get started. If you want to skip
   email confirmation while testing, turn off "Confirm email" under
   **Authentication → Settings**.

## 2. Point the app at your project

Open `index.html` and edit these two lines near the top of the `<script>`
block:

```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';
```

That's it — the app talks to Supabase directly from the browser via the
`@supabase/supabase-js` CDN script already included in `<head>`.

## How accounts work

- First launch shows a **Sign In / Sign Up** screen (email + password).
- New accounts get two starter recipes seeded automatically on first load,
  so the app isn't empty on day one.
- Recipes are stored in the `recipes` table, scoped to `user_id`. Row Level
  Security policies enforce that a signed-in user can only read, insert,
  update, or delete their *own* rows — this is enforced by Postgres, not
  just by the app's UI, so it holds even if someone calls the API directly.
- Tapping the circular initial badge in the top-right of the recipe index
  opens an account panel with **Sign Out**.
- Guest count is still kept locally per-device (`localStorage`), since it's
  a per-session cooking preference rather than account data.

## Deploying to GitHub Pages (taragourmet.4twenty7.com)

1. Push these files to `https://github.com/jmmsquared/TaraGourmet/`
2. In the repo settings, enable GitHub Pages from the branch you pushed to
3. Point the `taragourmet.4twenty7.com` CNAME at the Pages URL, and add a
   `CNAME` file to the repo containing `taragourmet.4twenty7.com`
4. In Supabase, add `https://taragourmet.4twenty7.com` to
   **Authentication → URL Configuration → Redirect URLs** (and Site URL) so
   auth flows work from the deployed domain
5. Because Pages can serve from a subpath, all asset references in this app
   are relative (`./manifest.json`, `./sw.js`) — don't change them to
   root-absolute paths or icons/offline support will 404
6. After any update, users who've added the app to their home screen need to
   delete and re-add the icon to pick up the new service worker

## iOS audio note

Timer chimes use the Web Audio API, unlocked on the very first tap anywhere
in the app (required by iOS Safari before any sound can play). If you test in
a desktop browser and hear nothing, click anywhere once first.

## Adding real app icons

`manifest.json` references `icon-192.png` and `icon-512.png`, which aren't
included — drop your own 192×192 and 512×512 PNGs (wildcat blue `#0033A0`
background recommended) into this folder before deploying, or the home-screen
icon will fall back to a browser default.

## Security notes

- The `anon` key is meant to be public/client-visible — access control comes
  from the Row Level Security policies in `schema.sql`, not from hiding the
  key. Never put the `service_role` key in this file.
- If you ever need to reset a user's data or manage accounts in bulk, do that
  from the Supabase dashboard (Table Editor / Authentication tab), not by
  loosening the RLS policies.
