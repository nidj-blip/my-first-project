---
name: video-pipeline
description: Handles Claruna's AI asset generation hand-off, FFmpeg rendering, and Cloudflare R2 storage — turning an AI-produced video structure into a finished video file. Use when a task requires writing or changing asset-generation calls, FFmpeg command construction, render worker logic, or media asset handling to/from Cloudflare R2.
---

# Video Pipeline Engineer (Claruna)

Handles FFmpeg rendering and Cloudflare R2.

## Activation conditions

Use this skill when a request requires:
- Calling an AI provider to generate assets (visuals, narration audio,
  music, captions) from the structure produced for a `generation_jobs` row.
- Constructing or changing an FFmpeg command (assembling generated assets
  into one video: transitions, caption burn-in/track, audio mix, encoding).
- Implementing or modifying the render worker that consumes
  `generating_assets`/`rendering`-stage jobs (per `CLARUNA_BLUEPRINT.md` §8).
- Handling media asset download from / upload to Cloudflare R2 as part of a
  generation job.

Do not use this skill for the job queue's API-facing endpoints (enqueue/poll
routes belong to `fastify-backend`, which advances job state and calls into
this skill's workers) or for mobile playback code (`expo-frontend`).

## Responsibilities

1. Implement the asset-generation and render stages exactly as described in
   `CLARUNA_BLUEPRINT.md` §8: generate each asset the AI structure calls
   for → upload to R2 and record in `generation_assets` → FFmpeg-assemble
   per the structure's timing/order → upload final video to R2 → update job
   status.
2. Keep FFmpeg invocations safe: never build a command string by
   concatenating unsanitized input (prompt text, AI-generated captions,
   filenames) — pass arguments as an argument array, and
   validate/allowlist any dynamic value (duration, asset id/path) before it
   reaches FFmpeg. The user's original prompt and any AI-generated text are
   untrusted input, per §10.
3. Run asset generation and rendering fully out-of-band from API
   request/response cycles — this skill's code runs in worker
   process(es), never inline in a Fastify route handler.
4. Clean up all local temp files after a job completes or fails, so repeated
   generations don't exhaust disk on the Railway service.
5. Produce output matching the Josnify.Ai dark-luxury-editorial brand
   direction (§3) where the pipeline has creative latitude (e.g. caption
   styling, transition choices).

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §3 (brand), §6 (data model:
   `generation_jobs`, `generation_assets`), and §8 (pipeline stages) before
   changing asset-generation or render logic.
2. Implement/modify the worker function that picks up a job at the
   `generating_assets` stage: call the AI provider(s) for each required
   asset kind, upload each to R2, insert a `generation_assets` row, and
   advance `status` to `rendering` once all assets are ready.
3. Implement/modify the render worker: build the FFmpeg command as an
   argument array (not a shell string), with every dynamic value (asset R2
   keys, durations, caption text) validated/allowlisted before use.
4. On success: upload the MP4 to R2, update `generation_jobs.status=ready`
   with `output_r2_key`; on failure at any stage: update
   `status=failed` with a stored `error_reason`, respecting the bounded-retry
   policy in §8.
5. Add a smoke test that runs the asset-generation-to-render pipeline
   against a fixture prompt/structure with mocked AI provider responses
   (no live paid API calls), to catch regressions in command construction
   and state transitions.

## Required quality checks

- [ ] FFmpeg is invoked with an argument array, never a shell-interpolated
      string built from prompt text, AI-generated captions, or any other
      user/AI-influenced value (command-injection risk).
- [ ] Every user- or AI-influenced parameter (duration, asset path, caption
      text) is validated/allowlisted before reaching FFmpeg or a file path.
- [ ] Temp files are cleaned up on both success and failure paths.
- [ ] Job status transitions (`generating_assets`→`rendering`→
      `ready`/`failed`) match §8 exactly, including bounded retries.
- [ ] A smoke test exercises the asset-generation-to-render pipeline with
      mocked AI responses and passes before the change is considered done.

## Blueprint alignment

The pipeline's stages, status enum, and brand direction come from
`CLARUNA_BLUEPRINT.md` §3 and §8. Any change to the pipeline itself (new
asset kind, new stage) should be reflected back into §8 via
`product-architect` if it changes the documented contract.

## Scope discipline

Touch only the asset-generation/render worker code and its direct tests.
Reuse existing code patterns; avoid unnecessary refactoring. Do not modify
the Fastify route layer's request/response contracts (coordinate with
`fastify-backend`) or mobile playback code.
