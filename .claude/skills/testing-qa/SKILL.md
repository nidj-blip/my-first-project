---
name: testing-qa
description: Tests Claruna features before release across Expo, Fastify, Supabase, and the AI-asset/FFmpeg render pipeline. Use when a task needs new tests written, an existing test suite run, or a QA pass after any implementation phase before it's considered done.
---

# QA Engineer (Claruna)

Tests features before release.

## Activation conditions

Use this skill:
- After every implementation phase, before marking it complete (per the
  project rule: test before committing, and run tests after every phase).
- When a request explicitly asks for tests to be written or a test suite to
  be run/fixed.
- When an implementation skill (`expo-frontend`, `fastify-backend`,
  `supabase-database`, `video-pipeline`) has produced a change that hasn't
  been verified yet.

## Responsibilities

1. Maintain the test pyramid described in `CLARUNA_BLUEPRINT.md` §10: unit
   tests for Fastify handlers and data-access functions, integration tests
   against a local/staging Supabase instance, Expo component/unit tests, and
   a full-pipeline smoke test (prompt → structure → assets → render) using
   fixture/mocked AI responses so it doesn't require live paid API calls.
2. Run the relevant suite(s) after every implementation phase and report
   pass/fail clearly — do not report a phase as complete on "should work."
3. When a test fails, diagnose root cause before deciding whether the fix
   belongs to the test or the implementation, and route it to the owning
   skill if it's implementation.
4. Keep tests fast and deterministic where possible — mock AI-provider calls
   rather than hitting live/paid endpoints in routine test runs; flag (don't
   silently skip) any test that's flaky or requires unavailable external
   services.

## Step-by-step workflow

1. Identify which layer(s) changed (mobile / API / database / video
   pipeline) from the phase just completed.
2. Run that layer's existing test command(s); if none exist yet for touched
   code, write focused tests covering the new behavior's happy path, at
   least one error/edge case, and (for auth/entitlement code) at least one
   unauthorized-access or quota-exhausted case.
3. For API changes: run integration tests against a local/dev Supabase
   instance if available; otherwise clearly note that integration coverage
   is pending environment setup.
4. For pipeline changes: run the full generation smoke test with mocked AI
   provider responses against a fixture prompt (see `video-pipeline`).
5. Report results: what ran, what passed/failed, and — for any failure —
   the concrete cause, not just "test failed."
6. Do not declare a phase done while any test it covers is failing or
   skipped without an explicit, stated reason.

## Required quality checks

- [ ] Every implementation phase has at least one test run associated with
      it before being marked complete.
- [ ] New auth/entitlement logic has a test proving unauthorized access and
      quota-exhausted submission are both rejected, not just that the
      authorized/in-quota path works.
- [ ] New RLS-dependent behavior has a test attempting cross-user access and
      a direct write to pipeline-internal fields, confirming both are
      denied.
- [ ] The full-pipeline smoke test (with mocked AI responses) passes on any
      change to `video-pipeline` or `fastify-backend` job-orchestration
      logic.
- [ ] No test is silently skipped/disabled to make a suite pass — skips are
      called out explicitly with a reason.

## Blueprint alignment

Test strategy and coverage expectations come from `CLARUNA_BLUEPRINT.md`
§10. If a phase's scope isn't yet reflected in the blueprint's
non-functional requirements, flag it rather than inventing ad hoc coverage
expectations.

## Scope discipline

Touch only test files and, where a failing test reveals an implementation
bug, the minimal fix in the owning layer (coordinating with that layer's
skill). Do not use a testing pass as cover for unrelated refactors.
