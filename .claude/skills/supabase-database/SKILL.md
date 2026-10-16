---
name: supabase-database
description: Designs Claruna's Supabase schema and Row Level Security policies — profiles, generation_jobs, generation_assets, entitlements. Use when a task requires creating or changing database tables/columns, RLS policies, or Supabase Auth configuration.
---

# Database Engineer (Claruna)

Designs Supabase schema and security.

## Activation conditions

Use this skill when a request requires:
- Creating, altering, or dropping a table/column in the Supabase Postgres
  database (e.g. `generation_jobs`, `generation_assets`, `entitlements`).
- Writing or changing a Row Level Security (RLS) policy.
- Supabase Auth configuration (providers, JWT settings) relevant to the app.
- Generating/updating TypeScript types from the database schema for use by
  `fastify-backend` and `expo-frontend`.

Do not use this skill to write API route handlers (`fastify-backend`) or
mobile client data-fetching code (`expo-frontend`) — this skill owns the
schema and policies those consume.

## Responsibilities

1. Keep the schema in sync with `CLARUNA_BLUEPRINT.md` §6 (Data Model) —
   every table/column there should exist in migrations, and every migration
   should be reflected back into §6 if it changes the model.
2. Write RLS policies so a user can only `select` their own
   `generation_jobs`/`generation_assets`/`entitlements` rows; only the
   Fastify API (service-role key) can `insert`/`update`
   `generation_jobs.status`, `structure`, and `generation_assets` — the
   user's own client must never write generation-internal state directly,
   per §6.
3. Never treat RLS as the only access control for entitlement/credit
   enforcement — that check also lives at the API layer
   (`fastify-backend`), per §10.
4. Manage migrations as versioned, forward-only SQL files (never hand-edit
   production schema outside a migration).

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §6 and the current migration history before
   proposing a schema change.
2. Write a new migration file (never modify an already-applied migration)
   for the change, including the RLS policy for any new table.
3. Apply the migration to a local/dev Supabase instance first; verify RLS
   behavior with both an authenticated test user and an anonymous/other
   user's session to confirm isolation, and confirm a user's own
   (non-service-role) session cannot write `generation_jobs.status` or
   `structure` directly.
4. Regenerate TypeScript types from the schema for consumption by
   `fastify-backend` / `expo-frontend`.
5. Update `CLARUNA_BLUEPRINT.md` §6 (via `product-architect` if the change is
   product-driven, or directly if it's a pure implementation detail already
   agreed) so the document and schema never drift.

## Required quality checks

- [ ] Every new table has an explicit RLS policy — no table is left with RLS
      disabled or a default-allow policy "for now."
- [ ] A test confirms a user cannot read another user's row, and cannot write
      `generation_jobs` pipeline-internal fields via the client (anon-key +
      user JWT) — only the service-role key on the API server can.
- [ ] Migrations are additive/forward-only; no editing of already-applied
      migration files.
- [ ] Generated TypeScript types are regenerated and committed alongside the
      schema change.
- [ ] `CLARUNA_BLUEPRINT.md` §6 matches the actual schema after the change.

## Blueprint alignment

`CLARUNA_BLUEPRINT.md` §6 is the canonical data model description. Schema
changes not yet reflected there should be routed through `product-architect`
first if they represent a product decision, not just a technical refinement.

## Scope discipline

Touch only migration files, RLS policy definitions, and generated type
files. Reuse existing code patterns; avoid unnecessary refactoring. Do not
modify API route logic or mobile app code — hand those changes to
`fastify-backend` / `expo-frontend` once the schema is in place.
