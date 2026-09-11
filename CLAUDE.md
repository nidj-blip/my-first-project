All project rules, architecture, workflows, and AI agent responsibilities for
Claruna live in **`CLARUNA_BLUEPRINT.md`** (source of truth) and
**`AGENTS.md`** (agent/skill role map).

Claruna is an AI video creation platform that generates videos from a single
emotional prompt.

Tech stack (fixed — do not introduce alternatives without an explicit product
decision recorded in `CLARUNA_BLUEPRINT.md`):
- Expo / React Native
- Fastify
- Supabase
- Railway
- Cloudflare R2
- FFmpeg

Rules:
- Follow `CLARUNA_BLUEPRINT.md` and `AGENTS.md`.
- Use the matching skill under `.claude/skills/` for the layer you're working
  in (product, design, frontend, backend, database, video pipeline, security,
  testing, deployment) — each skill's `SKILL.md` states its activation
  conditions, responsibilities, and required quality checks.
- Reuse existing code patterns; avoid unnecessary refactoring.
- Test before committing.
- Keep documentation (`CLARUNA_BLUEPRINT.md` in particular) updated when a
  product or architecture decision changes.
