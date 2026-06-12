# InvestClub — Deployment Guide

## What's included
- `index.html` — full app with Supabase backend (replaces localStorage)
- `vercel.json` — Vercel hosting config
- This README

---

## Step 1 — Create Free Supabase Database

1. Go to **https://supabase.com** → Sign up (free)
2. Click **New Project** — choose any name/password/region
3. Wait ~1 min for it to provision
4. Go to **Settings → API** and copy:
   - **Project URL** (looks like `https://abcdefgh.supabase.co`)
   - **anon/public** key (long JWT starting with `eyJ...`)

5. Go to **SQL Editor** in Supabase and run this SQL:

```sql
create table if not exists ic_users (
  id text primary key,
  mobile text,
  pw_hash text,
  name text,
  initials text,
  role text default 'member',
  joined_at text,
  needs_setup boolean default false,
  palette text,
  created_at timestamptz default now()
);
create table if not exists ic_contributions (
  id text primary key,
  user_id text,
  amount numeric,
  date text,
  note text,
  created_at timestamptz default now()
);
create table if not exists ic_trades (
  id text primary key,
  symbol text,
  name text,
  exchange text default 'NSE',
  type text,
  shares integer,
  price numeric,
  date text,
  note text,
  created_at timestamptz default now()
);
create table if not exists ic_prices (
  symbol text primary key,
  price numeric
);
create table if not exists ic_pricehistory (
  id bigserial primary key,
  symbol text,
  t bigint,
  price numeric
);
alter table ic_users enable row level security;
alter table ic_contributions enable row level security;
alter table ic_trades enable row level security;
alter table ic_prices enable row level security;
alter table ic_pricehistory enable row level security;
create policy "allow all" on ic_users for all using (true) with check (true);
create policy "allow all" on ic_contributions for all using (true) with check (true);
create policy "allow all" on ic_trades for all using (true) with check (true);
create policy "allow all" on ic_prices for all using (true) with check (true);
create policy "allow all" on ic_pricehistory for all using (true) with check (true);
```

---

## Step 2 — Deploy to Vercel (Free)

### Option A: Drag & Drop (easiest, no Git needed)
1. Go to **https://vercel.com** → Sign up free
2. From the dashboard, click **Add New → Project**
3. Click **"Deploy from a folder"** (or drag the folder)
4. Upload this entire folder (`investclub/`)
5. Click **Deploy** — done in ~30 seconds
6. Vercel gives you a free URL like `https://investclub-xyz.vercel.app`

### Option B: Via GitHub (auto-deploys on changes)
1. Push this folder to a GitHub repo
2. Connect repo to Vercel → it auto-deploys on every push

---

## Step 3 — First Launch

1. Open your Vercel URL
2. You'll see the **Connect Supabase** screen
3. Paste your Project URL and anon key → click Connect
4. You'll be taken to **Admin Setup** — set your admin mobile + password
5. Share the URL with club members — they log in with credentials you create in Admin → Members

---

## Notes
- **Free tiers**: Supabase free = 500MB DB + 50k row reads/day. Vercel free = unlimited static hosting.
- **Credentials are stored in sessionStorage** — members re-enter them once per browser session (or you can hardcode them in index.html for convenience).
- **Hardcoding credentials** (optional): Open `index.html`, find `function getSB()` and replace with:
  ```js
  function getSB(){
    return{url:"https://YOUR.supabase.co",key:"YOUR_ANON_KEY",ready:true};
  }
  ```
  This skips the connection screen entirely — good once you're set up.
- **Live prices** use Yahoo Finance via allorigins proxy — same as before, no change needed.
