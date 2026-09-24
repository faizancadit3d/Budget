# Splitly 💸

A budget tracker and friend expense-splitter — track your own monthly spending by category, and split shared costs (trips, rent, dinners) with friends, with live balances.

**Stack:** Next.js (App Router) + Tailwind CSS v4 + Supabase (Postgres, Auth, Realtime) + Vercel.

## Features

- **Email/password auth** via Supabase Auth, with protected routes handled in middleware.
- **Personal budgeting** — monthly limits per category, spend tracking, income vs. expense, custom categories.
- **Groups** — create a group or join one with an invite code.
- **Expense splitting** — split a cost equally, by exact amount, or by percentage.
- **Balances & settle up** — automatic net-balance calculation per member plus a minimal-transaction "who should pay who" suggestion; record payments to settle debts.
- **Realtime** — when a friend adds an expense or records a payment, your group page updates live (Supabase Realtime), no refresh needed.

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) → **New project**.
2. Once it's up, open **SQL Editor** → **New query**, paste the entire contents of [`supabase/schema.sql`](supabase/schema.sql), and run it. This creates all tables, RLS policies, triggers, the default categories, and enables Realtime on the relevant tables. It's safe to re-run.
3. Go to **Project Settings → API** and copy:
   - `Project URL`
   - `anon` `public` API key
4. Go to **Authentication → URL Configuration** and set:
   - **Site URL**: `http://localhost:3000` for local dev (change to your production domain after deploying).
   - **Redirect URLs**: add `http://localhost:3000/auth/callback` (and later `https://your-app.vercel.app/auth/callback`).
5. (Optional) **Authentication → Providers → Email**: decide whether "Confirm email" is on. If it's on (default), new users get a confirmation email before they can sign in — the app already handles both cases.

## 2. Configure environment variables

```bash
cp .env.local.example .env.local
```

Fill in the two values from step 1.3:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

## 3. Run locally

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000), sign up, and go.

## 4. Deploy to Vercel

1. Push this repo to GitHub (or GitLab/Bitbucket).
2. On [vercel.com](https://vercel.com), **Add New → Project**, import the repo.
3. Add the two environment variables from `.env.local` in the Vercel project settings (**Settings → Environment Variables**).
4. Deploy.
5. Back in Supabase → **Authentication → URL Configuration**, update the **Site URL** to your Vercel domain and add `https://your-app.vercel.app/auth/callback` to **Redirect URLs**.

## Project structure

```
supabase/schema.sql          Full DB schema: tables, RLS policies, triggers
src/middleware.ts            Session refresh + route protection
src/lib/supabase/            Browser / server / middleware Supabase clients
src/lib/data.ts              Read queries (groups, budgets, dashboard)
src/lib/actions/             Server Actions — all writes (auth, groups, expenses, budget)
src/lib/balances.ts          Net balance + debt-simplification math
src/app/(app)/               Authenticated shell: /dashboard, /groups, /budget
src/app/login, /signup       Auth pages
src/app/auth/callback        Email confirmation / magic-link handler
```

See [AGENTS.md](AGENTS.md) for more implementation notes.

## Data model at a glance

- `profiles` — mirrors `auth.users`, created automatically on sign-up.
- `categories` — shared defaults (`user_id is null`) plus per-user custom ones.
- `groups` / `group_members` — a group has an `invite_code`; joining looks it up via the `join_group_by_code` RPC.
- `expenses` / `expense_splits` — one expense, split across one row per participant.
- `settlements` — a recorded payment between two members of a group.
- `personal_budgets` / `personal_transactions` — per-user, per-month category limits and spending.

All tables have row-level security scoped to group membership or the owning user — see `supabase/schema.sql` for the exact policies.
