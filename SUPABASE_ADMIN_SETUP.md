# Duraclean Supabase Admin Panel Setup

This site currently has a lightweight local Admin Studio (`admin.html`) that is useful for demos, but it stores posts only in the browser. Use Supabase when Duraclean needs a real admin panel that is private, secure, and works across phones/laptops.

## What Supabase will add

- **Admin-only login:** only approved Duraclean staff can publish.
- **Permanent posts:** blogs, vlogs, and photos are stored in a database instead of one browser.
- **Image/video uploads:** before/after photos can be stored in Supabase Storage.
- **Website feed:** the landing page can load approved posts automatically.
- **Future tracking:** staff location/job updates can be stored in real time for an Uber-style tracker.

## Recommended setup

### 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com/).
2. Create a new project, for example `duraclean`.
3. Copy the **Project URL** and **anon public key** from **Project Settings → API**.
4. Keep the **service role key private**. Never paste it into `index.html` or `admin.html`.

### 2. Create the database tables

Open **SQL Editor** in Supabase and run:

```sql
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  email text not null,
  role text not null default 'viewer',
  created_at timestamptz not null default now()
);

create table public.posts (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  content text,
  type text not null check (type in ('blog', 'vlog', 'photo')),
  media_url text,
  published boolean not null default true,
  created_by uuid references auth.users(id),
  created_at timestamptz not null default now()
);

alter table public.profiles enable row level security;
alter table public.posts enable row level security;
```

### 3. Add row-level security policies

These policies let the public website read published posts, but only admins can create/edit/delete.

```sql
create policy "Public can read published posts"
on public.posts for select
using (published = true);

create policy "Admins can manage posts"
on public.posts for all
to authenticated
using (
  exists (
    select 1 from public.profiles
    where profiles.id = auth.uid()
    and profiles.role = 'admin'
  )
)
with check (
  exists (
    select 1 from public.profiles
    where profiles.id = auth.uid()
    and profiles.role = 'admin'
  )
);
```

### 4. Create the admin user

1. In Supabase, go to **Authentication → Users**.
2. Invite or create the Duraclean admin email.
3. After the user exists, run this SQL and replace the email:

```sql
insert into public.profiles (id, email, role)
select id, email, 'admin'
from auth.users
where email = 'admin@example.com'
on conflict (id) do update set role = 'admin';
```

### 5. Add media storage

1. Go to **Storage**.
2. Create a public bucket named `duraclean-media`.
3. Upload before/after photos, blog pictures, and vlog thumbnails there.
4. Save the public URL in the `posts.media_url` field.

### 6. Connect the website JavaScript

Add Supabase JS to `admin.html` and `index.html`:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
  const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';
  const SUPABASE_ANON_KEY = 'YOUR_ANON_PUBLIC_KEY';
  const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
</script>
```

Admin login example:

```js
const { data, error } = await supabase.auth.signInWithPassword({
  email: adminEmail,
  password: adminPassword
});
```

Publish post example:

```js
await supabase.from('posts').insert({
  title,
  content,
  type,
  media_url: mediaUrl,
  published: true
});
```

Website feed example:

```js
const { data: posts } = await supabase
  .from('posts')
  .select('title, content, type, media_url, created_at')
  .eq('published', true)
  .order('created_at', { ascending: false })
  .limit(6);
```

## Social media auto-posting

For Instagram/Facebook auto-display, use one of these:

- **Simplest:** an embed widget such as Elfsight, LightWidget, or SociableKIT.
- **Professional:** Meta Graph API connected to Supabase Edge Functions.
- **No-code automation:** Zapier or Make watches Instagram/Facebook and inserts new posts into Supabase.

Recommended non-technical flow:

1. Staff post photos or vlogs on Instagram/Facebook.
2. Zapier/Make detects the new post.
3. Automation creates a row in Supabase `posts`.
4. Website feed updates automatically.

## Future Uber-style tracking

For real tracking, add a `jobs` table and staff mobile check-ins:

```sql
create table public.jobs (
  id uuid primary key default gen_random_uuid(),
  client_name text not null,
  status text not null default 'assigned',
  cleaner_name text,
  latitude double precision,
  longitude double precision,
  eta_minutes int,
  updated_at timestamptz not null default now()
);
```

The staff phone can update `latitude`, `longitude`, `status`, and `eta_minutes`; the client page can subscribe to changes with Supabase Realtime.
