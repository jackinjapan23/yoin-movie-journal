# 余韻 — Yoin Cinema & Kotoba Journal

A responsive Japanese movie review and vocabulary journal built with React, TypeScript, Vite, Tiptap, and Supabase. The review editor stores a structured document; every highlighted occurrence has an inline UUID mark and a row in `vocabulary_occurrences`, linked to a reusable vocabulary definition. No journal content is stored as a local-only fallback.

## What is implemented

- Email/password sign-up and sign-in, persistent Supabase sessions, password reset, and sign-out.
- Film library with search, sorting, film creation, rating, poster URL, metadata, and review creation.
- Japanese rich-text review editor with headings, bold, italic, blockquotes, preview, manual save, and debounced autosave.
- Selection-based vocabulary creation, inline highlights, vocabulary sidebar, global searchable/film-filtered vocabulary list, learned state, editing, deletion, and exports.
- JSON exports for reviews and vocabulary; CSV export for vocabulary; individual review Markdown export.
- Responsive layouts, readable Japanese typography, loading/error states, and private row-level policies for every exposed table.

## Setup

1. Install Node.js 20.19+ or 22.12+ and pnpm, then install dependencies in this folder:

   ```sh
   pnpm install --frozen-lockfile
   ```

2. Create a Supabase project. In **Project Settings → API Keys**, copy the project URL and a **publishable** key. Never put a secret/service-role key in this app.

3. Copy `.env.example` to `.env.local` and fill in:

   ```dotenv
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_...
   ```

4. Apply the SQL migration. With the Supabase CLI installed, link the project and push migrations:

   ```sh
   supabase login
   supabase link --project-ref YOUR_PROJECT_REF
   supabase db push
   ```

   Or paste `supabase/migrations/202610270001_initial_journal.sql` into the Supabase SQL Editor. The migration creates the tables, indexes, update timestamps, grants, and ownership policies.

5. In **Authentication → URL Configuration**, set the local Site URL to `http://localhost:5173` while developing. Add your eventual production URL to the redirect allow list. Choose whether sign-up confirmation is required in **Authentication → Providers → Email**.

6. Start the site:

   ```sh
   pnpm dev
   ```

## Deploy to GitHub Pages

The included `.github/workflows/deploy-pages.yml` builds the app and publishes it whenever a commit is pushed to `main`. Its Supabase URL and publishable key are browser-side public configuration values; the site sends them to every visitor's browser as part of the app. Database access is protected by row-level security. Never add a Supabase secret/service-role key. Enable GitHub Pages with **Build and deployment → GitHub Actions** if it is not already enabled.

After the first deployment, set the Supabase Authentication **Site URL** to the deployed Pages URL, including its repository path (for example, `https://OWNER.github.io/REPOSITORY/`). This lets sign-up confirmation and password-reset links return to the hosted app.

## Backups

- Export review JSON and vocabulary JSON/CSV from the app's Settings and Vocabulary pages.
- For database-level backups, use the Supabase dashboard's **Project Settings → Database → Backups**. Backup availability and retention depend on the selected Supabase plan. For additional portable copies, periodically export both JSON files and store them separately from the hosted project.

## Data and design notes

- Reviews are saved as Tiptap JSON, preserving Japanese Unicode and inline vocabulary marks. Autosave is debounced by 900 ms and reports confirmation from Supabase.
- Deleting a film removes its review and occurrence rows through foreign-key cascades; vocabulary definitions remain in the user's collection. Deleting a vocabulary definition removes its occurrences, while review text remains.
- The occurrence identifier is stored in both the editor mark and occurrence table; this remains stable when prose before or after a mark is edited. Concurrent edits currently use last-write-wins autosave; version conflict warnings and automatic recovery for disconnected editing are not implemented yet.
- Word reading is always user-entered. No automatic kanji reading is inferred.
- The furigana control is currently a reading display option in the reading companion rather than an automatic linguistic ruby annotator for all kanji in the review.

## Current limitations

- Supabase project credentials and migration application are required before sign-in or cloud data can work. This workspace does not contain access to a user's Supabase account.
- Inline selected occurrences have stable marks. Editing/deleting words from the global vocabulary list updates the dictionary; review marks continue to display their saved mark until the review is edited to remove them.
- There is no server-side conflict resolution, poster upload storage, or automatic furigana generation.
