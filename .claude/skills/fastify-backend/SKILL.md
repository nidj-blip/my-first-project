---
name: fastify-backend
description: Builds Claruna's Fastify APIs — prompt submission, generation-job orchestration, entitlement/credit checks, and the AI-provider/render worker hand-off. Use when a task requires writing or modifying API routes, auth/JWT verification, entitlement logic, or the generation-job queue.
---

# Backend Engineer (Claruna)

Builds Fastify APIs.

## Activation conditions

Use this skill when a request requires:
- Adding or changing a Fastify route/handler (e.g. `POST /v1/generations`,
  `GET /v1/generations/:id`).
- Implementing auth (Supabase JWT verification) or entitlement/credit-quota
  logic.
- Implementing or modifying the generation-job queue/state machine that
  drives the structure → assets → render pipeline (§8), including calls out
  to the AI provider(s) and hand-off to the `video-pipeline` skill's FFmpeg
  logic.
- Server-side integration with Supabase (service-role) or Cloudflare R2
  (signed URLs, uploads).

Do not use this skill for mobile client code (`expo-frontend`), schema/RLS
changes (`supabase-database`, though this skill consumes that schema), or
FFmpeg command construction itself (`video-pipeline` owns that logic; this
skill owns the job orchestration around it).

## Responsibilities

1. Implement the API surface defined in `CLARUNA_BLUEPRINT.md` §7, keeping
   routes, request/response shapes, and auth requirements in sync with that
   document.
2. Verify every non-public request's Supabase-issued JWT server-side; never
   trust a client-supplied user id.
3. Enforce entitlement/generation-credit checks before enqueuing a job — per
   §7 and §10, quota is checked at submission time, not only before
   rendering, since AI calls and rendering both cost money per job.
4. Own the generation-job state machine (`queued` → `structuring` →
   `generating_assets` → `rendering` → `ready`/`failed`) described in §8,
   including bounded retries and clear failure reasons.
5. Treat the user's prompt as untrusted input: validate/sanitize before
   passing it to an AI provider call, and never let prompt content reach a
   shell command or FFmpeg argument string directly (that's
   `video-pipeline`'s concern, but this skill must not bypass it).
6. Hold the only credentials that can call the AI provider(s), write to R2,
   or use the Supabase service-role key; never let those leak into logs,
   error responses, or the mobile client.

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §4–8 (stack, architecture, data model, API
   surface, generation pipeline) before adding or changing a route.
2. Confirm the route's auth requirement and which entitlement/credit check
   (if any) gates it.
3. Implement the handler with input validation (Fastify schema validation
   for request bodies/params — including prompt length/content checks, not
   ad hoc string checks).
4. For anything touching the generation pipeline: enqueue/advance state via
   the `generation_jobs` table described in §6/§8; never perform AI-provider
   calls or FFmpeg work synchronously inside a request handler.
5. Write unit tests for the handler's auth/entitlement/validation branches,
   and an integration test against a local/staging Supabase instance where
   feasible.
6. Run lint/typecheck/test for the API package before considering the change
   done.

## Required quality checks

- [ ] Every non-public route verifies the Supabase JWT server-side.
- [ ] `POST /v1/generations` checks `entitlements.generation_credits` before
      enqueuing, not just before rendering.
- [ ] Request bodies/params (including the prompt itself) are
      schema-validated; no unvalidated input reaches a database query, an AI
      provider call, or a file-system/FFmpeg call.
- [ ] No AI-provider call or FFmpeg/render work runs synchronously inside a
      request/response cycle — it goes through the job state machine in §8.
- [ ] No secret (service-role key, R2 write credentials, AI provider API key)
      appears in logs, error messages, or API responses.
- [ ] Lint, typecheck, and test commands for the API package pass.

## Blueprint alignment

The API surface, auth model, and generation-job state machine in
`CLARUNA_BLUEPRINT.md` §5, §7, §8 are binding. If a requested change isn't
covered there, update the blueprint (via `product-architect`) before
implementing, rather than silently diverging from the documented contract.

## Scope discipline

Touch only the API server directory and files directly required for the
requested route/feature. Reuse existing code patterns; avoid unnecessary
refactoring. Do not modify mobile app code, database migration files
(coordinate with `supabase-database` instead), or unrelated routes.
