# Claruna Agent Guide

Each role below is implemented as a skill under `.claude/skills/<name>/SKILL.md`.
Read `CLARUNA_BLUEPRINT.md` first — it is the source of truth every skill
must follow. This file is the quick role map; the skill files hold the full
activation conditions, step-by-step workflow, and quality checks.

- **Product Architect** (`product-architect`) — plans features and the
  roadmap, owns the data model and API surface in the blueprint.
- **UI/UX Designer** (`ui-ux-designer`) — designs screens and user flows in
  the Josnify.Ai dark-luxury-editorial brand.
- **Frontend Engineer** (`expo-frontend`) — builds the Expo / React Native
  mobile app.
- **Backend Engineer** (`fastify-backend`) — builds the Fastify API and
  generation-job orchestration.
- **Database Engineer** (`supabase-database`) — designs the Supabase schema
  and Row Level Security policies.
- **Video Pipeline Engineer** (`video-pipeline`) — handles AI asset
  generation hand-off, FFmpeg rendering, and Cloudflare R2 storage.
- **Security Reviewer** (`security-review`) — cross-cutting review of auth,
  secrets, and input handling across every layer above.
- **QA Engineer** (`testing-qa`) — tests features before release; runs the
  test suite after every implementation phase.
- **Deployment Engineer** (`deployment`) — manages Railway, Supabase
  environments, Cloudflare R2 buckets, and EAS mobile builds.

## Workflow (confirmed product loop)

1. User enters an emotional prompt (Expo/React Native app).
2. Backend creates a generation job (Fastify + Supabase).
3. AI creates a video structure from the prompt.
4. Assets are generated (visuals, narration, music, captions).
5. FFmpeg renders the video from the generated assets.
6. The rendered video is uploaded to Cloudflare R2.
7. The user receives the completed video.

## Rules

- Follow `CLARUNA_BLUEPRINT.md`.
- Reuse existing code patterns.
- Test before committing.
- Keep documentation updated.
- Avoid unnecessary refactoring.
