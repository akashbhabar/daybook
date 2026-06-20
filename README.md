# 📖 Daybook

**A daily memory journal that lives in one HTML file — Instagram-style stories, mood tracking, photo memories, and free cloud sync, with zero backend to deploy.**

> Capture today. Relive any day. Never lose a memory.

---

## ✨ What is this?

Daybook is a single-file, mobile-first web app for logging daily memories — short journal entries with a title, mood, photos, tags, location, and a "how was it" rating. It's built to feel like a polished native app (think Instagram meets a journaling app), but it's just one `.html` file you can open in any browser, host anywhere, or hand to a friend.

No build step. No `npm install`. No server required to get started — open the file and it works.

---

## 🪄 Features

### 📝 Memory logging
- Dedicated **title** and **description** fields (not auto-merged)
- 5-point **mood scale** — Amazing, Good, Okay, Bad, Terrible — each with its own color
- Photo & video attachments (auto-compressed client-side before storage)
- Tags, people, location, date/time, and a "rate this memory" slider with a live emoji reaction
- Quick-formatting toolbar (bullets, quotes, emoji)

### 🎬 Stories — like Instagram, but for your life
A horizontal story rail on Home, auto-built from your real data:
- **Today's Memory**
- **This Week's Best** (highest-rated entry since Monday)
- **On This Day** (same date, past years)
- **Most Loved** (anything you've liked)
- Recent-day highlights

Tap to open a full-screen 9:16 story viewer — photo background, memory text overlaid, auto-advancing progress bar, swipe/tap navigation, keyboard arrows, swipe-down to close.

### 🗓️ Five core screens
| Screen | What it does |
|---|---|
| **Home** | Today's entries, week strip with mood dots, stories rail |
| **Timeline** | Chronological feed grouped by month/day |
| **Calendar** | Month grid with mood-colored dots + day detail sheet |
| **Insights** | Monthly trend, 30-day mood chart, top tags |
| **Profile** | Stats, streak, account, backup, and settings |

### 📱 Real responsive design
Mobile-first with a bottom tab bar; automatically becomes a sidebar-driven desktop layout at wider viewports — no separate "desktop version," same file.

### ☁️ Cloud sync (Supabase) + offline-first
- Email **one-time-code** sign-in (no magic links, no redirect headaches — works from a local file or any host)
- Memories sync to a Postgres table; photos upload to Supabase Storage
- **Local-first**: every change saves to `localStorage` instantly, then syncs in the background — the app keeps working offline and catches up when reconnected
- One-time automatic migration of any memories you made before signing in
- Optional secondary backup to a Google Sheet (text only) via a simple Apps Script

### 🔒 Privacy & safety details
- Row Level Security in Postgres — you can only ever read/write your own data
- **Clear all memories** requires typing a confirmation phrase before the delete button even activates — no accidental wipes
- Export everything to JSON at any time

---

## 🚀 Getting started

### Option A — Just open it
Double-click `daily-memory-log.html`, or open it in any browser. It works immediately, saving to your browser's local storage. No account needed — tap **"Continue without an account"** on the sign-in screen.

### Option B — Host it anywhere
It's one static file, so any static host works: GitHub Pages, Netlify, Vercel, S3, your own server. Just upload it.

### Option C — Turn on cloud sync (recommended for multi-device use)
1. Create a free [Supabase](https://supabase.com) project
2. Run the table + storage bucket setup (SQL provided below)
3. Drop your Project URL and `anon` key into the `CONFIG` section at the top of the script
4. Open the app, sign in with email, done

<details>
<summary><strong>Supabase setup SQL</strong> (click to expand)</summary>

```sql
-- Memories table
create table memories (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users not null,
  title text, content text, mood text,
  date date not null, time text, rating int,
  tags text[] default '{}', people text[] default '{}', location text,
  liked boolean default false, bookmarked boolean default false,
  media jsonb default '[]', deleted boolean default false,
  created_at timestamptz default now(), updated_at timestamptz default now()
);

alter table memories enable row level security;

create policy "Users can manage their own memories"
on memories for all
using (auth.uid() = user_id)
with check (auth.uid() = user_id);

-- Storage policies for the `memory-photos` bucket (create the bucket in the dashboard first)
create policy "Users can upload their own photos"
on storage.objects for insert
with check (bucket_id = 'memory-photos' and auth.uid()::text = (storage.foldername(name))[1]);

create policy "Users can view their own photos"
on storage.objects for select
using (bucket_id = 'memory-photos' and auth.uid()::text = (storage.foldername(name))[1]);

create policy "Users can delete their own photos"
on storage.objects for delete
using (bucket_id = 'memory-photos' and auth.uid()::text = (storage.foldername(name))[1]);
```

**Don't forget:** in **Authentication → Email Templates**, make sure the template includes `{{ .Token }}` so sign-in emails contain a code instead of a link.
</details>

---

## 🛠️ Tech stack

Deliberately minimal — no framework, no bundler, no package.json:

- **Vanilla HTML / CSS / JavaScript** — the entire app is one file
- **[Supabase](https://supabase.com)** — Postgres database, Auth (email OTP), and Storage, all on the free tier
- **`localStorage`** — offline-first cache and fallback when no account is connected
- **Plus Jakarta Sans** (Google Fonts) for type

No React, no Vue, no build tooling. If you can open an HTML file, you can run this.

---

## 📁 Project structure

```
daily-memory-log.html   ← the entire app: markup, styles, and logic in one file
```

That's it. That's the whole structure.

---

## 🎨 Design philosophy

Soft, modern UI with rounded cards, gradient accents, and a calm blue/purple palette — designed to feel inviting for a daily habit, not clinical like a form. Mobile-first because journaling happens in spare moments, usually on a phone.

---

## 🗺️ Possible next steps

Ideas for where this could go, not promises:
- IndexedDB for larger local photo caches before cloud sync kicks in
- Story reactions / replies
- Shareable read-only memory links
- Dark mode
- Video stories with playback controls

---

## 📜 License

Personal project — use it, fork it, make it yours.

---

*Built one conversation at a time. Every pixel and policy was iterated on, tested, and fixed for real bugs — not just described.*
