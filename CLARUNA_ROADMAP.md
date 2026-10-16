# Claruna — Repository Audit & Implementation Roadmap

Companion to `CLARUNA_BLUEPRINT.md` (source of truth for product/architecture)
and `AGENTS.md` (role map). This file is the audit-then-roadmap deliverable:
it does not begin implementation — per project rule, major implementation
starts only after this audit and roadmap are reviewed.

---

## 1. Repository Audit (current state vs. blueprint)

**Current full file inventory of `nidj-blip/my-first-project`:**

```
.claude/settings.json                  # claude-mem plugin marketplace registration only
.claude/skills/*/SKILL.md              # 9 skills (this task's deliverable)
AGENTS.md                              # agent/role map (this task's deliverable)
CLAUDE.md                              # project pointer file (this task's deliverable)
CLARUNA_BLUEPRINT.md                   # source of truth (this task's deliverable)
README.md                              # "learning github" personal note, unrelated to Claruna
```

**Finding: there is no application code of any kind in this repository.** No
`package.json`, no Expo project, no Fastify server, no Supabase migrations, no
FFmpeg/worker code, no tests, no CI/CD config, no `.env.example`. The repo's
git history (4 commits) contains only the README and a plugin-marketplace
registration — nothing was ever built here. This is a from-scratch build, not
a migration or refactor of existing code.

This matches the "Do not overwrite working code unnecessarily" instruction
trivially: there is no working code to protect yet, so every phase below is
net-new, and the "avoid unnecessary refactoring" rule applies from Phase 1
onward as code accumulates.

## 2. Missing Components by Category

| Category | Blueprint reference | Status |
|---|---|---|
| **Frontend (Expo/React Native)** | §5, §7, §8 | Missing entirely — no Expo project scaffolded |
| **Backend (Fastify)** | §5, §7, §8 | Missing entirely — no API server scaffolded |
| **Database (Supabase)** | §6 | Missing entirely — no project, no migrations, no RLS policies |
| **Rendering (AI asset gen + FFmpeg)** | §8 | Missing entirely — no worker process, no AI provider integration |
| **Testing** | §10 | Missing entirely — no test runner configured for any layer |
| **Deployment (Railway/EAS/R2)** | §9 | Missing entirely — no Railway project, no EAS config, no R2 bucket setup, no CI |

## 3. Blocking Open Questions

These items from `CLARUNA_BLUEPRINT.md` §11 block specific phases below and
must be resolved by the owner before that phase can start for real (a
placeholder/mock can unblock earlier phases in the meantime):

- **AI provider(s)** for structure generation and asset generation (blocks
  Phase 4/5 real integration; Phase 4/5 can proceed against a mocked provider
  interface until resolved).
- **Monetization model** (credits vs. subscription) — blocks the exact shape
  of Phase 8, not earlier phases (a generic `generation_credits` column
  already accommodates either).
- **Content moderation policy** — should be resolved before Phase 9
  (production deployment), not before earlier phases.
- **Target video spec** (aspect ratio, duration, caption style) — blocks
  `ui-ux-designer` and `video-pipeline` detail work in Phase 3/6, not the
  scaffolding phases before them.

---

## 4. Phased Implementation Roadmap

Each phase is small, independently testable, and ends with `testing-qa`
running before moving on, per project rule. No phase begins until the
previous phase's tests pass.

### Phase 0 — Planning (this task, complete)
- `CLARUNA_BLUEPRINT.md`, `CLAUDE.md`, `AGENTS.md`, 9 skills, this roadmap.
- Test gate: none (no code yet) — review gate instead: owner confirms the
  corrected product understanding (§1 of the blueprint) and the open
  questions in §11 are acceptable to proceed against as-is.

### Phase 1 — Repository Scaffolding
- Monorepo layout (e.g. `apps/mobile` for Expo, `apps/api` for Fastify,
  shared `packages/types` for cross-layer TypeScript types).
- Bare Expo app boots (`expo start`), bare Fastify server boots with a health
  check route, linting/formatting/typecheck configured for both.
- Test gate: `testing-qa` runs lint + typecheck + a trivial "server boots and
  health check returns 200" test, and an Expo build/smoke check.

### Phase 2 — Supabase Foundation
- Supabase project (local dev instance), `profiles` table + Auth wiring,
  baseline RLS policy pattern established and proven with a test.
- Generated TypeScript types wired into `apps/api` and `packages/types`.
- Test gate: `supabase-database` cross-user RLS test passes;
  `testing-qa` confirms auth sign-up/sign-in works end to end against local
  Supabase.

### Phase 3 — Prompt Submission & Job Creation
- `generation_jobs` table + RLS (owner-write-only for pipeline fields, per
  §6).
- `POST /v1/generations` and `GET /v1/generations/:id` on Fastify, JWT-guarded,
  no AI/FFmpeg work yet — job is created and immediately stubbed to
  `status=ready` with a placeholder value, to prove the request→job→poll loop.
- Mobile prompt-entry screen (per `ui-ux-designer` spec) wired to submit and
  poll.
- Test gate: end-to-end test — submit a prompt from a test client, poll to
  completion, confirm RLS prevents reading another user's job.

### Phase 4 — AI Structure Generation
- Real (or provider-agnostic mocked, if §11's open question is still
  unresolved) call turning a prompt into the `structure` jsonb described in
  §6/§8; job advances `queued→structuring→generating_assets` (assets stage
  stubbed for now).
- Test gate: unit tests for the structure-call wrapper (including a
  moderation/validation check on the returned structure); pipeline smoke test
  with a mocked provider passes.

### Phase 5 — Asset Generation
- Worker generates each asset kind from the structure (visuals, narration,
  music, captions), uploads to R2, records `generation_assets` rows; job
  advances to `rendering`.
- Test gate: smoke test with mocked asset-generation calls confirms all
  expected `generation_assets` rows exist with valid R2 keys before advancing
  state.

### Phase 6 — FFmpeg Rendering
- `video-pipeline` FFmpeg assembly of generated assets into one output video,
  argument-array invocation with validated inputs (per `security-review`'s
  standing check), upload to R2, job reaches `ready`.
- Test gate: full pipeline smoke test (Phase 3–6 combined) with fixture
  assets and mocked AI calls produces a valid output file; `security-review`
  confirms no shell-string FFmpeg construction.

### Phase 7 — Mobile Result Experience
- Generation-progress UI across all real pipeline states, finished-video
  review/download/share screen, error and quota-exhausted states.
- Test gate: `expo-frontend` component tests for every state; manual/dev-client
  run through the full golden path plus a forced-failure path.

### Phase 8 — Entitlements & Credits
- `entitlements` table, credit deduction on job submission, refund on
  definitive failure (policy per §8's open question — default to "refund on
  failure" unless the owner specifies otherwise), store-webhook stub.
- Test gate: tests proving credit deduction, refund-on-failure, and
  quota-exhausted rejection all work; `security-review` confirms the check
  happens at submission time, not just before rendering.

### Phase 9 — Deployment
- Railway staging deploy (API + workers), Supabase staging project, R2
  staging bucket, EAS preview build.
- Test gate: `deployment` skill's checklist (environment confirmed,
  `testing-qa` green, rollback path known) before promoting to production
  configuration.

---

## 5. What Happens Next

Per the task's own instructions, **major implementation does not start until
this roadmap and the audit above are reviewed** by the owner. The next step
is owner sign-off (or corrections) on:
1. The corrected product understanding in `CLARUNA_BLUEPRINT.md` §1.
2. This phase breakdown and its test gates.
3. Which open question(s) in §11 must be answered before Phase 4 vs. which
   can proceed against a mock.

Once confirmed, Phase 1 (repository scaffolding) is the first implementation
work, done in isolation and tested before Phase 2 begins.
