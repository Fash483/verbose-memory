# ✦ Magnet Vault ✦

A React + Vite torrent magnet-link manager, backed directly by **Supabase**
(Postgres + auto-generated REST API), deployed as a static site on
**Netlify**. The frontend talks to Supabase straight from the browser using
the `@supabase/supabase-js` client — no custom backend server required.

## Quick start

```bash
npm install
cp .env.example .env   # paste your Supabase URL + anon key
npm run dev             # Vite dev server on :5173
```

## 1. Set up Supabase

1. Create a project at [supabase.com](https://supabase.com).
2. Open the **SQL Editor** and run the contents of `supabase/schema.sql`.
   This creates the `links` table and Row Level Security policies that allow
   public read/insert/delete via the anon key (suitable for a single-user /
   personal vault — see the note in that file if you want to lock it down
   with Supabase Auth instead).
3. Go to **Project Settings → API** and copy the **Project URL** and
   **anon public key**.

## 2. Configure environment variables

Copy `.env.example` to `.env` and fill in:

```
VITE_SUPABASE_URL=https://your-project-ref.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

These are safe to expose to the browser — they're the public anon key, and
access is controlled by the RLS policies in `supabase/schema.sql`. Never put
the `service_role` key in frontend code.

## 3. Deploy to Netlify

**Option A — Netlify UI**

1. Push this repo to GitHub/GitLab/Bitbucket.
2. In Netlify: **Add new site → Import an existing project**, pick the repo.
3. Build command: `npm run build`, publish directory: `dist` (already set
   in `netlify.toml`).
4. Under **Site settings → Environment variables**, add `VITE_SUPABASE_URL`
   and `VITE_SUPABASE_ANON_KEY`.
5. Deploy.

**Option B — Netlify CLI**

```bash
npm install -g netlify-cli
netlify login
netlify init
netlify env:set VITE_SUPABASE_URL "https://your-project-ref.supabase.co"
netlify env:set VITE_SUPABASE_ANON_KEY "your-anon-key"
netlify deploy --prod
```

`netlify.toml` already configures the build command, publish directory, and
an SPA redirect (`/* -> /index.html`) so client-side routing works.

## Data model

Table: `public.links`

| Column     | Type        |
| ---------- | ----------- |
| id         | bigint (PK) |
| title      | text        |
| magnet     | text        |
| created_at | timestamptz |

## What changed vs. the Fly.io/Postgres version

- Removed the Express server (`server/index.js`) and Dockerfile/`fly.toml` —
  there's no custom backend anymore.
- The frontend now calls Supabase directly via `@supabase/supabase-js`
  (see `src/supabaseClient.ts`), instead of hitting a same-origin `/api/*`
  Express API.
- Deployment target: Netlify static hosting (`netlify.toml`) instead of a
  Fly.io container.
- Database: Supabase Postgres (with RLS policies) instead of Fly Postgres /
  plain `pg`.
