<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# Splitly — project notes

Budget tracker + friend expense-splitter. Next.js (App Router) + Tailwind v4 + Supabase (Postgres, Auth, Realtime). See [README.md](README.md) for setup.

- `supabase/schema.sql` — full DB schema, RLS policies, triggers. Source of truth for the data model; `src/lib/database.types.ts` is hand-written to match it (no generated types, since there's no live project wired into this repo).
- `src/lib/supabase/{client,server,middleware}.ts` — browser/server/middleware Supabase clients (`@supabase/ssr`). `src/middleware.ts` refreshes the session and gate-keeps routes.
- `src/lib/data.ts` — read queries for groups, budgets, dashboard. `src/lib/actions/*.ts` — all mutations as Server Actions (`"use server"`).
- `src/lib/balances.ts` — net-balance + debt-simplification math shared between the group page and dashboard.
- Route groups: `(app)` wraps the authenticated shell (`NavBar` + auth check) around `/dashboard`, `/groups`, `/budget`. `/`, `/login`, `/signup` sit outside it.
- Realtime: `src/components/RealtimeWatcher.tsx` subscribes to `postgres_changes` on a group's tables and calls `router.refresh()` so balances/expenses update live for everyone in the group.
- Money is `numeric(12,2)` in Postgres and plain `number` (dollars) in TS — `round2()` guards against float drift before writes.
