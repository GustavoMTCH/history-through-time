# History Through Time V3

A public, read-only historical timeline with a private invited-editor workspace. The app preserves the V2 timeline experience and moves its events, eras and images into a shared Supabase database and Storage bucket. Realtime subscriptions refresh open copies when another editor changes content.

## Access model

- **Visitors:** no login; can explore the timeline and view event/era images.
- **Invited collaborators:** authenticate by email magic link; after an administrator grants the `editor` role, they can add, edit and delete events and upload shared images.
- **Admins:** same editor rights, plus can administer collaborator roles through Supabase.

## 1. Create the Supabase project

1. Create a project at https://supabase.com/dashboard.
2. Open **SQL Editor → New query**.
3. Paste and run the complete `supabase/schema.sql` file.
4. In **Authentication → Providers → Email**, enable email sign-in and email OTP/magic links. Disable public sign-ups (the app requests sign-in for existing invited accounts only).
5. In **Authentication → URL Configuration**, add the eventual Vercel URL to the allowed redirect URLs (for local testing, add `http://localhost:3000`).
6. In **Project Settings → API**, copy the Project URL and the **anon/public** key.

The SQL creates the tables, row-level security policies, image bucket, starter events and eras, and Realtime publication membership.

## 2. Configure the app

Edit `config.js` and replace the two placeholders:

```js
export const SUPABASE_URL = 'https://YOUR-PROJECT-REF.supabase.co';
export const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_PUBLIC_KEY';
```

Use only the anon/public key in the browser. **Never place a `service_role` or secret key in `config.js`, HTML, Git, or Vercel client-side variables.** The anon key is designed to be public; database and Storage permissions are enforced by the RLS policies in the SQL file.

## 3. Create the first administrator

1. In Supabase **Authentication → Users**, invite your own email address.
2. Open the invitation/sign-in email and sign in to the deployed app once. This creates your `profiles` row with the safe default role `viewer`.
3. In Supabase SQL Editor, promote your account (replace the email):

```sql
update public.profiles
set role = 'admin'
where lower(email) = lower('your-email@school.edu');
```

4. Sign out and sign back in. The **Edit history** button will now appear.

## 4. Invite colleagues as editors

1. In Supabase **Authentication → Users → Invite user**, invite each colleague by email.
2. Each colleague opens their invitation and signs in through **Editor sign in** on the public site.
3. Once they have signed in once, grant editor access in SQL Editor:

```sql
update public.profiles
set role = 'editor'
where lower(email) = lower('colleague@school.edu');
```

They must sign out and sign in again to refresh their role. Do not grant editor access to people who should only view the timeline.

## 5. Publish on Vercel

1. Put this folder in a Git repository (GitHub is easiest).
2. Import the repository at https://vercel.com/new.
3. Framework preset: **Other**; build command: leave blank; output directory: `.`; install command: leave blank.
4. Deploy. Add the resulting domain to Supabase **Authentication → URL Configuration → Redirect URLs**.
5. Open the public Vercel URL. Anyone can explore; only invited users with the editor role can change shared data.

You can also test locally by serving this folder with any static server (for example `npx serve . -l 3000`) and adding `http://localhost:3000` to Supabase redirect URLs.

## What is shared

- Event titles, dates, descriptions, categories and era labels.
- Era color palettes and background images.
- Event images and era background images, stored in Supabase Storage.
- Edits made by one collaborator appear on other open sessions through Supabase Realtime.

## Important safeguards

- Public visitors can only read rows and images.
- Only authenticated users whose `profiles.role` is `editor` or `admin` can write to events, eras or the media bucket.
- New invited accounts default to `viewer`; an administrator must explicitly promote them.
- The public anon key is not a password. RLS is the security boundary. Keep the `service_role` key private.
- Before going live, verify that public sign-up is disabled and that the SQL policies were applied successfully.

## Current V3 scope

This version provides shared editing of events, event images, and era background images; email-invited collaborator access; public viewing; and live refresh. It does not yet include simultaneous co-editing of the same form, comments, per-event approval workflows, or a visual admin screen for inviting users/assigning roles. Those can be added in a later iteration.
