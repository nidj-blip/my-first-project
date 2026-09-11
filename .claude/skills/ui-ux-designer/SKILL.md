---
name: ui-ux-designer
description: Designs Claruna's screens and user flows in the Josnify.Ai dark-luxury-editorial brand — the prompt-entry experience, generation progress states, and the finished-video review/share screen. Use when a task needs screen/flow design, wireframes, component visual specs, or a design review before or alongside Expo implementation.
---

# UI/UX Designer (Claruna)

Designs screens and user flows.

## Activation conditions

Use this skill when a request:
- Asks for a new screen, flow, or component to be designed (wireframe,
  layout, visual spec) before it's built in Expo.
- Asks for a design review of an existing screen against brand and usability
  standards.
- Needs a design decision (navigation pattern, progress/loading treatment for
  a multi-minute generation job, empty/error/quota-exhausted states,
  accessibility treatment) that implementation skills shouldn't invent
  unilaterally.

Do not use this skill to write React Native code — hand finished specs to
`expo-frontend` for implementation.

## Responsibilities

1. Keep every screen consistent with the Josnify.Ai dark-luxury-editorial
   brand (`CLARUNA_BLUEPRINT.md` §3): dark backgrounds, high-contrast
   serif/sans pairing, warm/gold accent, generous whitespace — no generic
   Material/iOS defaults, no generic "AI generator" clichés.
2. Design for the actual product loop (§1): a single emotional-prompt input,
   a generation-in-progress experience spanning multiple pipeline stages
   (queued → structuring → generating assets → rendering → ready), and a
   finished-video review/download/share screen — not a generic app template.
3. Specify every screen state explicitly: the prompt-entry happy path, each
   meaningful generation-progress state, error/failed-generation, offline,
   and quota-exhausted (no generation credits left) states.
4. Design for mobile-first, both iOS and Android idioms via Expo, including
   safe-area handling and dynamic type/accessibility sizing.

## Step-by-step workflow

1. Read `CLARUNA_BLUEPRINT.md` §1, §3, and §8 (product concept, brand,
   generation pipeline stages) and confirm the screen/flow being designed
   maps to a real pipeline stage or feature there; if it doesn't, flag it to
   `product-architect` before designing.
2. List the states the screen must support: happy path, each progress state
   the pipeline can be in, failed/error (with the failure reason surfaced
   plainly, not a generic "something went wrong"), offline, and
   quota-exhausted.
3. Produce the design spec: layout structure, typography scale, color roles
   (background/surface/accent/text-on-dark), spacing, and interaction notes
   (how progress through §8's pipeline stages is communicated without
   over-promising a completion time the backend can't guarantee).
4. Note any new reusable component this introduces (e.g. a prompt input
   card, a generation-progress tracker, a video result card) so
   `expo-frontend` can build it once and reuse it.
5. Call out accessibility requirements: minimum contrast on dark
   backgrounds, touch target sizes, screen-reader labels for icon-only
   controls and for progress-state announcements.

## Required quality checks

- [ ] Every state (happy path/each progress stage/error/offline/
      quota-exhausted) is specified, not just the ideal case.
- [ ] Progress communication matches the real pipeline stages in §8 — no
      fabricated percentage/ETA the backend doesn't actually provide.
- [ ] Visual direction matches §3 of the blueprint — flag any deviation
      explicitly rather than quietly introducing a different look.
- [ ] Contrast and touch-target sizes meet mobile accessibility norms on dark
      backgrounds specifically.
- [ ] Reusable components are named and specified once, not redesigned ad hoc
      per screen.
- [ ] The design fits what Expo/React Native can implement natively or with
      already-approved libraries.

## Blueprint alignment

Always design against the product scope, brand rules, and pipeline stages in
`CLARUNA_BLUEPRINT.md`. If a request implies a feature not in the blueprint,
route it to `product-architect` to get scoped and written down first.

## Scope discipline

Only produce design specs/assets for the screen or flow requested. Do not
modify backend code, database schema, or unrelated skills, and do not
redesign existing screens that weren't part of the request without calling
that out as a separate, explicit proposal first.
