# Care — Courses & Training Platform

Internal replacement for Hotmart, built for Grupo Care Anestesia's corporate training
(~70 users, no sales/checkout flow). Full background and architecture decisions in
[`LLMS - Care.md`](./LLMS%20-%20Care.md); phased development plan in
[`plano-desenvolvimento.md`](./plano-desenvolvimento.md); chronological build history
(only up to Phase 9) in [`historico-desenvolvimento.md`](./historico-desenvolvimento.md);
feature-parity audit against Hotmart in [`auditoria-hotmart.md`](./auditoria-hotmart.md).
None of those three documents is fully up to date with what came after they were last
written — treat them as historical record, not as the source of truth for the current
state of the code.

> User-facing UI and copy are all in Portuguese (the client's language) — only this
> README and the codebase docs are in English.

## Stack

- **Frontend**: Next.js 16 (App Router) + TypeScript + Tailwind CSS v4 — this version has
  breaking changes vs. older Next.js knowledge; see the notice at the top of
  [`AGENTS.md`](./AGENTS.md) before making framework-level changes.
- **Backend/database**: Supabase — Postgres + Auth (email/password) + Storage, all behind
  Row Level Security scoped by `empresa_id` (multi-tenant-ready, single real tenant today)
- **Video**: Cloudflare Stream (direct-from-browser upload, resumable via TUS for large
  files; progress tracking through the player)
- **Certificates**: pdf-lib, generated server-side
- **Rich text**: Tiptap (lesson support text editor)
- **Email**: SendGrid as Supabase Auth's custom SMTP, plus a separate SendGrid API
  integration for admin-triggered account emails (see [`AGENTS.md`](./AGENTS.md) for why)
- **Deploy**: Vercel

## Design system — "Care Terra"

Tokens in [`src/app/globals.css`](./src/app/globals.css) (Tailwind v4 `@theme inline`),
components in [`src/design-system`](./src/design-system) (atoms/molecules/organisms),
MD3-like styling. Check for an existing token before hardcoding a color or typography value.

## Local setup

```bash
npm install
# create .env.local with Supabase, Cloudflare Stream and SendGrid credentials
npm run dev
```

Without the Supabase environment variables filled in, the app still runs (the auth proxy
becomes a no-op), but any screen that depends on a logged-in session won't work.

## Database

`supabase/schema.sql` is the base schema (used only to recreate the database from scratch
via `supabase/reset.sql`). The rest of the database history lives in
`supabase/migrations/NNN_*.sql`, applied **manually** in the Supabase SQL Editor — there is
no automated migration runner wired up for this project. `schema.sql` has never been
updated retroactively with the migrations, so it is not an accurate picture of the current
schema on its own; read the migrations in order for that.

```bash
npm run build    # always run before pushing anything schema/build sensitive
npm run lint
npx tsc --noEmit
```

## Folder structure

```
src/
  app/                 routes (App Router)
  design-system/       atoms / molecules / organisms
  lib/supabase/        clients (browser, server, admin)
  lib/certificado.ts   PDF certificate generation
  lib/email.ts         SendGrid API integration
  proxy.ts             Supabase session refresh (Next.js 16 "proxy" convention)
supabase/
  schema.sql           base schema + RLS (not current — see above)
  reset.sql            recreates the database from schema.sql
  migrations/          NNN_*.sql, applied manually, source of truth for current schema

```
