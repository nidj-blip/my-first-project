# Claruna — Product & Architecture Blueprint

> **Status: v0.2 — corrected against owner-supplied project files.**
> v0.1 of this document was an assumption-based placeholder (no blueprint
> existed anywhere in this repo or its git history). It has been corrected
> using the owner-supplied `CLAUDE.md` / `AGENTS.md` / `README.md` from the
> actual Claruna project. Sections still marked `⚠️ OPEN QUESTION` are not yet
> confirmed and must not be treated as final. This file is the single source
> of truth for every `.claude/skills/*` file and all implementation work —
> skills and code must be re-aligned whenever this file changes.

---

## 1. What Claruna Is

**Confirmed** (from the project's own `CLAUDE.md`):

> Claruna is an AI video creation platform that generates videos from a
> single emotional prompt.

A user provides one emotional prompt (e.g. a feeling, a moment, a short piece
of text describing what they want to express). Claruna turns that single
input into a finished short video, end to end, without the user doing any
manual editing:

1. **User enters an emotional prompt** in the Expo/React Native mobile app.
2. **Backend creates a generation job** (Fastify API + Supabase row).
3. **AI creates a video structure** from the prompt — a script/scene
   breakdown (e.g. narration lines, scene descriptions, mood/music
   direction, pacing).
4. **Assets are generated** for that structure (visuals, narration audio,
   music/background, captions).
5. **FFmpeg renders the video** — assembling generated assets into a single
   output file (transitions, captions burned in or as a track, audio mix).
6. **Upload to Cloudflare R2** — the rendered video is stored as the
   durable, servable asset.
7. **User receives the completed video** in the app (preview, download,
   share).

This is the full product loop; there is no separate manual editing surface
in v1 — the emotional prompt is the entire creative input from the user.

Brand family: Claruna is a Josnify.Ai product, alongside the *Sacred
Frequency* YouTube channel and the book *Mia*. ⚠️ **OPEN QUESTION**: exact
relationship between Claruna's generated videos and the Sacred
Frequency/Mia content pipelines (e.g. does Claruna reuse Sacred Frequency's
audio/visual asset library, or is it fully independent?) is not yet
documented — confirm with the owner before an implementation depends on it.

## 2. Agent / Role Map (confirmed, from `AGENTS.md`)

The project's own agent guide defines these roles, which this repository's
`.claude/skills/*` implement 1:1 (plus two roles the owner's task added
explicitly — security review and a named testing/QA skill):

| AGENTS.md role | This repo's skill |
|---|---|
| Product Architect | `product-architect` |
| UI/UX Designer | `ui-ux-designer` |
| Frontend Engineer (Expo/React Native) | `expo-frontend` |
| Backend Engineer (Fastify) | `fastify-backend` |
| Database Engineer (Supabase schema + security) | `supabase-database` |
| Video Pipeline Engineer (FFmpeg + Cloudflare R2) | `video-pipeline` |
| QA Engineer | `testing-qa` |
| Deployment Engineer (Railway) | `deployment` |
| *(added: cross-cutting security review beyond DB-level security)* | `security-review` |

Project rules from `CLAUDE.md` / `AGENTS.md`, binding on every skill:
- Follow this blueprint (`CLARUNA_BLUEPRINT.md`) and `AGENTS.md`'s role
  definitions.
- Reuse existing code patterns; avoid unnecessary refactoring.
- Test before committing.
- Keep documentation updated (this file, in particular, must stay in sync
  with reality).

## 3. Brand & Design Constraints

- Brand family: **Josnify.Ai** (parent), sub-brands **Sacred Frequency**,
  **Mia**, **Claruna**.
- Visual direction: dark-luxury-editorial (consistent with the brand's other
  social/creative assets) — dark backgrounds, high-contrast serif/sans
  pairing, warm/gold accent, generous whitespace. Avoid generic Material/iOS
  defaults and stock "AI generator" clichés.
- Tone: the generated videos are emotionally expressive and personal — the
  app UI should feel calm and premium, not like a generic utility.

## 4. Technology Stack (owner-specified, fixed)

| Layer | Technology | Notes |
|---|---|---|
| Mobile app | **Expo (React Native)** | iOS + Android from one codebase; EAS Build for store binaries |
| API server | **Fastify** (Node.js/TypeScript) | Deployed on Railway; owns generation-job orchestration |
| Database & Auth | **Supabase** (Postgres + Auth + Row Level Security) | Source of truth for users, prompts, generation jobs, assets, entitlements |
| Object storage | **Cloudflare R2** | Generated assets (images/audio/video clips) and final rendered videos |
| Media rendering | **FFmpeg** (server-side, invoked from Fastify-orchestrated jobs) | Final assembly of generated assets into one video |
| AI structure/asset generation | External AI provider(s) — ⚠️ **OPEN QUESTION**: which model/vendor for (a) prompt → video structure and (b) asset generation (image/voice/music) is not yet specified by the owner | Called server-side only; API keys never reach the client |
| Hosting (API) | **Railway** | Fastify service + worker process(es) for generation/render jobs |
| Hosting (mobile) | **Expo Application Services (EAS)** | Build & OTA updates |

Constraints this implies:
- The mobile app never holds server secrets: no Supabase service-role key,
  no R2 write credentials, no AI provider API key. It talks to Supabase
  directly only for anon-key, RLS-scoped operations (auth, reading its own
  rows); everything privileged goes through the Fastify API.
- AI structure generation, asset generation, and FFmpeg rendering are all
  time-consuming and potentially costly — they run as async, queued jobs,
  never inline in a request/response cycle.

## 5. High-Level Architecture

```
┌─────────────────────┐        HTTPS (REST)        ┌──────────────────────────┐
│   Expo / React        │ ─────────────────────────▶ │  Fastify API (Railway)   │
│   Native app           │ ◀───────────────────────── │                          │
│  (iOS / Android)       │                             │  - auth guard (Supabase │
└─────────┬──────────────┘                             │    JWT verification)     │
          │  direct (anon key, RLS-scoped)              │  - generation-job        │
          ▼                                             │    endpoints             │
┌─────────────────────┐                                 │  - AI structure call     │
│      Supabase          │ ◀────────── service role ────▶│  - asset generation call │
│  Postgres + Auth        │                              │  - render orchestration  │
│  Row Level Security     │                              │  - entitlement/quota     │
└─────────────────────┘                                 └────────────┬─────────┘
                                                                      │
                                              ┌───────────────────────┼───────────────────────┐
                                              ▼                       ▼                       ▼
                                     ┌──────────────┐        ┌──────────────┐        ┌──────────────────┐
                                     │  AI provider(s) │        │  FFmpeg worker │        │  Cloudflare R2    │
                                     │  (structure +    │        │  (render step)  │───────▶│  (assets + final  │
                                     │   asset gen)     │───────▶│                 │        │   videos)          │
                                     └──────────────┘        └──────────────┘        └──────────────────┘
```

## 6. Data Model (initial, Supabase/Postgres)

⚠️ Refine once the AI provider(s) and asset types are confirmed by the owner.

```
profiles              (id uuid PK = auth.users.id, display_name, avatar_url, created_at)

generation_jobs        (id, user_id FK -> profiles, prompt text,
                        status enum [queued, structuring, generating_assets,
                                     rendering, ready, failed],
                        structure jsonb null,        -- AI-produced script/scene breakdown
                        error_reason text null,
                        output_r2_key text null,      -- final rendered video
                        requested_at, completed_at)

generation_assets       (id, job_id FK -> generation_jobs, kind enum
                        [image, voice_audio, music, caption_track],
                        source enum [ai_generated, stock, uploaded],
                        r2_key, position, created_at)

entitlements            (id, user_id FK -> profiles, plan enum [free, premium],
                        generation_credits int,       -- quota for AI/render cost control
                        source [store, promo], expires_at)
```

RLS baseline: every table with a `user_id` is scoped so a user can only
`select` their own rows; only the Fastify API (service-role key) can
`insert`/`update` `generation_jobs.status`, `structure`, and
`generation_assets` — a user's own client never writes generation-internal
state directly.

## 7. API Surface (Fastify, initial)

- `POST /v1/generations` — submit an emotional prompt, enqueue a
  `generation_jobs` row (`status=queued`), return its id.
- `GET /v1/generations/:id` — poll job status/progress; once `ready`, returns
  a signed R2 URL for the final video.
- `GET /v1/generations` — list the current user's past generations.
- `GET /v1/me/entitlements` — current plan and remaining generation credits.
- `POST /v1/webhooks/store` — App Store/Play Store server notifications →
  update entitlements.

All endpoints require a valid Supabase-issued JWT, verified server-side.
`POST /v1/generations` additionally checks `entitlements.generation_credits`
before enqueuing (cost control — each job triggers paid AI calls + compute).

## 8. Generation & Rendering Pipeline

1. Client calls `POST /v1/generations` with the prompt → Fastify validates
   input (length/content), checks entitlement/quota, enqueues job
   (`status=queued`), returns immediately.
2. **Structure worker** picks up `queued` jobs → calls the AI provider to turn
   the prompt into a structured script/scene breakdown → stores it in
   `structure`, sets `status=generating_assets`.
3. **Asset worker** generates each asset the structure calls for (narration
   audio, visuals, music selection) → uploads each to R2, records a row in
   `generation_assets`, sets `status=rendering` once all assets are ready.
4. **Render worker** (`video-pipeline` skill) invokes FFmpeg to assemble the
   assets per the structure's timing/order into one output video → uploads
   to R2, sets `status=ready` with `output_r2_key`.
5. Any stage failure sets `status=failed` with `error_reason`, subject to a
   bounded retry policy; the user's generation credit is refunded/not
   consumed on a definitive failure. ⚠️ **OPEN QUESTION**: exact refund/retry
   policy not yet specified by the owner.
6. Client polls (or is pushed a notification) and fetches the signed R2 URL
   once `ready`.

## 9. Environments & Deployment

- **Local dev**: Supabase CLI local stack (or a dev project) + Fastify via
  watch mode + Expo dev client/Expo Go; AI provider calls against a
  sandbox/test key where the provider supports one.
- **Staging**: Railway staging service + dedicated Supabase project + R2
  bucket prefix `staging/`.
- **Production**: Railway production service + production Supabase project +
  R2 bucket prefix `prod/`.
- Mobile builds ship via EAS (`development`, `preview`, `production` build
  profiles); OTA updates for JS-only changes where store review isn't
  required.

## 10. Non-Functional Requirements

- **Security**: no server secret (Supabase service-role key, R2 write
  credentials, AI provider API keys) ever reaches the mobile bundle; every
  privileged operation is behind the Fastify API with JWT verification;
  signed, time-limited R2 URLs only; user prompts are treated as untrusted
  input into both the AI provider call and any rendering step (no prompt
  content is ever interpolated into a shell command or FFmpeg argument
  string — see `video-pipeline` and `security-review`).
- **Cost control**: AI structure/asset generation and FFmpeg rendering all
  cost money/compute per job — entitlement/credit checks happen before a job
  is enqueued, not only before rendering.
- **Content safety**: ⚠️ **OPEN QUESTION** — what moderation (if any) applies
  to user-submitted emotional prompts and AI-generated output before it's
  shown to the user or is shareable, is not yet specified by the owner and
  must be resolved before a public launch.
- **Testing**: unit tests for Fastify route handlers and data-access
  functions; integration tests against a local/staging Supabase instance;
  Expo component/unit tests; a smoke test for the full generation pipeline
  (prompt → structure → assets → render) using fixture/mocked AI responses
  so it doesn't require live paid API calls on every run.
- **Observability**: structured logging on the Fastify service (request id,
  user id, job id, pipeline stage) so a stuck/failed job can be traced across
  workers.

## 11. Open Questions for the Owner

1. Which AI provider(s) power (a) prompt → video structure and (b) asset
   generation (image/voice/music)? This blocks concrete implementation of
   the structure and asset workers.
2. What is the monetization model — credits, subscription, both?
3. What content moderation applies to prompts and generated output?
4. How does Claruna relate to the existing Sacred Frequency /
   Mia content pipelines — shared asset library, fully independent, or a
   promotional cross-link only?
5. Target video spec: aspect ratio(s), duration range, caption style —
   needed by `ui-ux-designer` and `video-pipeline` before final implementation.

Until these are answered, every skill below treats this document as the
best-available source of truth but must flag (not silently resolve) any task
that depends on one of these open questions.
