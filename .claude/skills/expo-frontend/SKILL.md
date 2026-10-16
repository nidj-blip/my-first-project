---
name: expo-frontend
description: Builds Claruna's Expo and React Native mobile app — the emotional-prompt entry screen, generation-progress UI, and finished-video review/share screen. Use when a task requires writing or modifying mobile app screens, navigation, state management, or client-side calls to Supabase/the Fastify API.
---

# Frontend Engineer (Claruna)

Builds Expo and React Native features.

## Activation conditions

Use this skill when a request requires:
- Building or modifying a screen, navigator route, or reusable component in
  the Expo/React Native app.
- Wiring client-side data fetching to Supabase (anon-key, RLS-scoped reads,
  auth) or to the Fastify API (submitting a prompt, polling generation
  status, fetching the finished video).
- Client-side state management, form handling, or offline/loading/error UI
  behavior for the mobile app.

Do not use this skill to design new screens from scratch without a spec (use
`ui-ux-designer` first) or to write backend/API code (`fastify-backend`) or
database migrations (`supabase-database`).

## Responsibilities

1. Implement screens/components exactly to the design spec produced by
   `ui-ux-designer`, in the Josnify.Ai dark-luxury-editorial brand.
2. Keep the client thin and honest about the stack split in
   `CLARUNA_BLUEPRINT.md` §5: the app talks to Supabase directly only for
   anon-key, RLS-scoped operations (auth session, reading its own
   `generation_jobs`/`entitlements` rows); it talks to the Fastify API for
   anything privileged (submitting a prompt at `POST /v1/generations`,
   entitlement checks). Never embed a Supabase service-role key, R2 write
   credentials, or an AI provider API key in the mobile bundle.
3. Handle every UI state the screen spec calls for, including each
   generation-pipeline progress state (§8), failure, offline, and
   quota-exhausted.
4. Keep TypeScript types for API/Supabase responses accurate and shared
   where practical, so a backend contract change is caught at compile time.

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §4–8 (stack constraints, architecture, data
   model, API surface, pipeline stages) and the relevant `ui-ux-designer`
   spec for the screen.
2. Confirm which calls are Supabase-direct (anon key, RLS-scoped) vs.
   Fastify-API (JWT-authenticated) per §5 — get this wrong and either
   secrets leak or RLS becomes the only defense, which the blueprint
   explicitly treats as insufficient alone.
3. Implement the screen/component, matching the design spec's states and the
   brand's visual language, including polling/subscribing to
   `GET /v1/generations/:id` for progress until `status=ready` or `failed`.
4. Add/update TypeScript types for any new API or Supabase response shape
   used (e.g. `generation_jobs.status` enum values).
5. Write or update component/unit tests for new logic (state transitions
   across pipeline stages, conditional rendering for quota-exhausted,
   error handling).
6. Run the project's lint/typecheck/test commands for the mobile app before
   considering the change done.

## Required quality checks

- [ ] No server secret (Supabase service-role key, R2 write credentials, AI
      provider API key) is ever referenced from mobile app code.
- [ ] Prompt submission and entitlement checks go through the Fastify API,
      not directly to Supabase/R2/an AI provider from the client.
- [ ] Every pipeline progress state, plus error/offline/quota-exhausted, is
      implemented, not just the happy path.
- [ ] Lint, typecheck, and test commands for the mobile app pass.
- [ ] New reusable components match ones specified by `ui-ux-designer` rather
      than one-off duplicates of existing components.

## Blueprint alignment

Follow `CLARUNA_BLUEPRINT.md` for the Supabase-vs-Fastify split, the data
shapes in §6, and the API contracts in §7. If an API endpoint or data field
needed doesn't exist yet in the blueprint, flag it to `product-architect` /
`fastify-backend` rather than inventing a client-only workaround.

## Scope discipline

Touch only the mobile app directory and files directly required for the
requested screen/feature. Reuse existing code patterns; avoid unnecessary
refactoring. Do not modify backend, database migration, or infrastructure
files, and do not refactor unrelated screens as part of an unrelated task.
