---
name: deployment
description: Manages Claruna's Railway deployments, Supabase environments, Cloudflare R2 buckets, and EAS mobile builds. Use when a task requires deploying, configuring environments, or setting up CI/CD for any part of the stack.
---

# Deployment Engineer (Claruna)

Manages Railway deployments.

## Activation conditions

Use this skill when a request requires:
- Deploying or configuring the Fastify API (and its worker processes for
  asset generation/rendering) on Railway.
- Setting up or promoting Supabase environments (local → staging →
  production) and running migrations against them.
- Configuring Cloudflare R2 buckets/credentials per environment.
- Configuring AI provider API keys/credentials per environment, once §4's
  open question on provider choice is resolved.
- Building or submitting the Expo app via EAS (development/preview/
  production profiles).
- Setting up or changing CI/CD pipelines for any of the above.

Do not use this skill to write the application code being deployed — that
belongs to `expo-frontend` / `fastify-backend` / `supabase-database` /
`video-pipeline`.

## Responsibilities

1. Maintain the three-environment model in `CLARUNA_BLUEPRINT.md` §9: local
   dev, staging, production — each with its own Supabase project, R2 bucket
   prefix, and (where applicable) AI provider key/sandbox mode, never
   sharing credentials across environments.
2. Ensure the Fastify service and its workers on Railway only ever hold the
   service-role key, R2 write credentials, and AI provider keys for their
   own environment — staging never touches production data, buckets, or
   billed AI usage.
3. Manage EAS build profiles (development/preview/production) and OTA
   update channels so JS-only changes can ship without a full store review
   when appropriate, and native changes go through the correct build
   profile.
4. Keep deployment reversible: know how to roll back a Railway deploy and how
   to identify the last-known-good EAS build/OTA update.

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §9 (Environments & Deployment) to confirm
   which environment is the deploy target and what's expected to differ
   between environments.
2. Before deploying: confirm `testing-qa` has run and passed the relevant
   suite(s) for the change being shipped — never deploy an unverified change
   to production.
3. For a database change: run the migration against the target
   environment's Supabase project only, never against another environment's
   project.
4. For an API/worker change: deploy the Fastify service and its worker
   process(es) to Railway, then verify health (basic smoke check against a
   known-safe endpoint, and that a test generation job can progress through
   at least the `queued`→`structuring` transition) before considering the
   deploy done.
5. For a mobile change: pick the correct EAS build profile for the change's
   nature (JS-only → OTA-eligible; native dependency change → new build),
   and verify the build/submission completes.
6. Record what was deployed where (environment, version/commit, migration
   applied) so a rollback has a clear reference point.

## Required quality checks

- [ ] The target environment is explicitly confirmed before any deploy or
      migration — no ambiguity about staging vs. production.
- [ ] `testing-qa` has passed for the change before it's deployed to
      production.
- [ ] Environment credentials (Supabase service-role key, R2 keys, AI
      provider keys) are never shared or copied across environments.
- [ ] A rollback path is known and stated for the deploy performed (previous
      Railway deploy, previous EAS build/OTA channel state).
- [ ] Database migrations are applied to exactly the intended environment's
      Supabase project.

## Blueprint alignment

Environment topology and deployment targets come from
`CLARUNA_BLUEPRINT.md` §9. Any new environment or infrastructure change
should be reflected back into that section so future deploys have an
accurate reference.

## Scope discipline

Touch only deployment configuration, CI/CD pipeline definitions, and
environment-specific config — not application source code, unless a deploy
script itself is the artifact being changed. Do not deploy or migrate an
environment that wasn't the explicit target of the request.
