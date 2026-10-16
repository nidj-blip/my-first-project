---
name: security-review
description: Cross-cutting security review for Claruna across the Expo/Fastify/Supabase/R2/FFmpeg/AI-provider stack. Use before merging any change touching auth, entitlements/credits, RLS policies, secrets, prompt/AI input handling, or FFmpeg command construction, and whenever explicitly asked for a security review.
---

# Security Reviewer (Claruna)

## Activation conditions

Use this skill when:
- A change touches authentication, JWT verification, entitlement/credit
  logic, RLS policies, or any credential (Supabase service-role key, R2
  keys, AI provider API keys, signed URLs).
- A change handles user-supplied input (the emotional prompt, or any
  AI-generated text/caption derived from it) that reaches a database query,
  a file path, an AI provider call, or an FFmpeg command.
- A change is about to be merged/deployed and no security pass has happened
  yet.
- The user explicitly asks for a security review.

This skill reviews and reports; it does not replace the implementation
skills' own responsibility to write secure code in the first place.

## Responsibilities

1. Verify the secret/privilege boundaries in `CLARUNA_BLUEPRINT.md` §5 and
   §10 are actually upheld in code: mobile app holds no server secrets; only
   the Fastify API holds the Supabase service-role key, R2 write
   credentials, and AI provider API keys.
2. Verify every privileged/credit-gated API route checks auth (valid
   Supabase JWT) and entitlement/quota server-side — never trusts a
   client-supplied user id or credit balance.
3. Verify RLS policies exist and correctly isolate user-owned rows, and that
   a user's own client cannot write `generation_jobs` pipeline-internal
   state (`status`, `structure`) directly — only the service-role key can
   (spot-check independently of `supabase-database`'s own checks).
4. Verify the emotional prompt and any AI-generated text are treated as
   untrusted throughout: never interpolated into a shell command or FFmpeg
   argument string, never trusted to bound cost (e.g. an unbounded prompt
   length driving unbounded AI/render cost) without a validated limit.
5. Verify R2 access is via short-lived signed URLs, never public write
   access or long-lived credentials handed to the client.
6. Check for secrets or PII leaking into logs, error responses, or committed
   files (env files, sample config with real keys).

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §5 (architecture/privilege split) and §10
   (non-functional security/cost-control requirements) as the standard to
   review against.
2. Identify the diff/change under review; map each touched file to the layer
   it belongs to (mobile / API / database / video pipeline).
3. For each layer, run the layer-specific checks above (auth boundary, RLS,
   prompt/AI-input handling, FFmpeg input handling, secret exposure, cost
   control).
4. Trace at least one end-to-end privileged flow (e.g. submitting a prompt
   through to a completed generation) through client → API → AI provider →
   FFmpeg → R2 → database to confirm no step trusts an unverified claim or
   unbounded input.
5. Report findings ranked by severity (secret exposure / auth bypass /
   unbounded-cost input first), each with the concrete failure scenario, not
   just a generic "looks fine" or "consider adding validation."
6. If a finding requires a fix, hand it to the owning skill
   (`fastify-backend`, `supabase-database`, `video-pipeline`,
   `expo-frontend`) rather than patching outside this skill's scope.

## Required quality checks

- [ ] No server secret (service-role key, R2 write credentials, AI provider
      API key) appears in mobile app code, client bundles, or committed
      config.
- [ ] Every privileged/credit-gated route verifies JWT + entitlement
      server-side.
- [ ] RLS policies exist on every user-owned table and were checked against
      an actual cross-user access attempt and a direct-write attempt on
      pipeline-internal fields, not just read from the migration.
- [ ] Prompt/AI-generated text never reaches a shell command or FFmpeg
      argument string unsanitized; prompt length/content is bounded before
      it can drive unbounded AI or render cost.
- [ ] R2 access uses short-lived signed URLs; no public write bucket policy.
- [ ] No secret/PII appears in logs or error responses reviewed.

## Blueprint alignment

The privilege/trust boundaries this skill enforces come directly from
`CLARUNA_BLUEPRINT.md` §5 and §10. If a reviewed change requires a boundary
the blueprint doesn't yet describe, flag it to `product-architect` rather
than approving an undocumented trust model.

## Scope discipline

This skill reports findings and, when explicitly asked, applies narrowly
scoped fixes to the specific issue found — it does not refactor unrelated
code, redesign features, or touch files outside the change under review.
