# Product pipeline: idea → shipped

> Referenced from root `CLAUDE.md`. How a raw idea becomes shipped code.

A raw idea becomes shipped code through a fixed pipeline. **Stage 0 runs once**
over the whole idea; **stages 1–5 run per epic**.

## Stage 0 — Epic split (once)

| Command | Does |
| --- | --- |
| `/slice` | Turns a brain-dump into scoped, independently shippable epics as job stories |

Run this before anything else on a multi-part idea. Its job is to force
"is this actually one thing or two?" and — critically — **to kill overlaps**.
A brain-dump written in sections almost always has two sections touching the
same logic; find that here, not after two PRDs have been written against it.

## Stages 1–5 — per epic

| # | Stage | Command | Output |
| --- | --- | --- | --- |
| 1 | Clarify | `/grill-me` | Open questions and half-finished logic resolved |
| 2 | Specify | `/spec-writer` | `docs/prd/NNN-slug.md` |
| 3 | **Adversarial gate** | `/grill-me` on the PRD | Contradictions, gaps, scope creep |
| 4 | Slice to backlog | `/slice-the-spec` | Vertical slices, dependency-ordered, HITL/AFK flagged |
| 5 | Split if still big | `/story-splitting` | Sprint-sized tickets |

Then build.

## Rules

- **Stage 3 is a gate, not a formality.** Grill the *written PRD*, not the idea
  again — the point is to find what the document got wrong. Do not proceed to
  stage 4 while anything critical is open. Record the pass as `grilled: true` in
  the PRD frontmatter, and only then move `status` to `approved`.
- **Stage 1 must clear every `TO INVESTIGATE` / open item** before stage 2. An
  unresolved unknown that reaches the PRD becomes an unresolved unknown in the
  backlog.
- **Stage 5 is conditional.** `/slice-the-spec` already produces vertical
  slices. Only run `/story-splitting` on tickets that still smell too big — it
  keys off red flags like "and", "or", "manage", "handle". If a story has no
  obvious split point but is still too large, use `/hamburger-method` instead
  (generates options per layer, composes minimal end-to-end slices).
- **`/grill-me` and `/slice` are user-invoked only** (`disable-model-invocation:
  true`). You cannot trigger them. When a stage calls for one, **stop and tell
  the user to run it, then wait.**
  - Do **not** ask the stage's questions yourself instead. That has already
    happened once (2026-09-06): four rounds of ad-hoc Q&A stood in for stages 0
    and 1. The answers were useful, but `/slice` and `/grill-me` were never run,
    so the design-tree discipline and the frontier-per-round structure were both
    lost.
  - The drift is gradual and each step looks reasonable — the user asks a
    question, you answer it, they ask another. Notice the *pattern*, not the
    individual turn. `grilling` is the model-invocable implementation behind
  `grill-me` if the behaviour is needed directly.
- **Stages 0, 1 and 3 are conversations; only 2, 4 and 5 produce artifacts.**
- **This pipeline is expensive at scale.** N epics costs roughly 4N interactive
  passes. For a large idea, confirm with the user whether to run all epics or
  sequence the highest-risk ones first — do not silently start a 40-pass run.
- **For a genuinely trivial change**, say so and ask whether to skip the
  pipeline rather than skipping it silently.

## PRDs

PRDs live in `docs/prd/` and are the **source of truth for feature intent**.

- Read `docs/prd/README.md` for conventions (numbering, frontmatter, lifecycle).
- PRDs are produced by `/spec-writer` (step 2 above), written to
  `docs/prd/NNN-slug.md`. `docs/prd/TEMPLATE.md` mirrors that skill's
  10-section structure, so there is exactly one template — keep them in sync if
  the skill is updated.
- **Before implementing any feature, read its PRD.** If a non-trivial feature
  has no PRD, say so and offer to run the pipeline.
- Only implement against PRDs with status `approved` or `in-progress`. A `draft`
  PRD means requirements are still moving — flag it rather than building on it.
- If implementation reveals that a requirement is wrong, incomplete, or
  contradictory, update the PRD and tell the user. Do not silently diverge from
  it in code.
- Archive with `git mv docs/prd/NNN-slug.md docs/prd/_archive/` — never delete.
