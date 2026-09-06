# PRDs

> Auto-loaded when Claude reads anything in `docs/prd/`.

Product Requirement Documents for this project. A PRD is the **source of truth for
feature intent** — what we are building and why. It is not a design doc and not a
task list.

## Layout

```
docs/prd/
├── README.md          # this file
├── TEMPLATE.md        # copy this to start a new PRD
├── NNN-slug.md        # active PRDs
└── _archive/          # shipped or dropped PRDs
```

## Naming

`NNN-kebab-slug.md` — zero-padded 3-digit number, monotonically increasing,
never reused. Example: `001-appointment-booking.md`, `002-reminder-emails.md`.

Find the next number with:

```bash
ls docs/prd/*.md docs/prd/_archive/*.md 2>/dev/null | grep -oE '[0-9]{3}' | sort -n | tail -1
```

## Frontmatter

Every PRD starts with:

```yaml
---
id: 001
title: Appointment booking
status: draft        # draft | approved | in-progress | shipped | dropped
owner: nayan.jagetiya@allego.com
created: 2026-09-06
updated: 2026-09-06
related: []          # other PRD ids, issue links, docs
grilled: false       # true once the PRD has passed the /grill-me adversarial pass
---
```

## Lifecycle

| Status | Meaning |
| --- | --- |
| `draft` | Being written. Requirements still moving. Do not implement. |
| `approved` | Signed off. Safe to design and implement against. |
| `in-progress` | Implementation underway. |
| `shipped` | Delivered. Move to `_archive/`, keep status. |
| `dropped` | Abandoned. Move to `_archive/`, add a `## Why dropped` section. |

Archiving is a `git mv` into `_archive/` — never delete a PRD. The trail of
rejected ideas is worth more than the disk space.

## How a PRD gets built

PRDs are produced by a fixed skill pipeline. Do not hand-write one from scratch
unless the pipeline is unavailable.

| Stage | Command | Produces |
| --- | --- | --- |
| 0. Epic split (once) | `/slice` | Scoped, deduplicated epics as job stories |
| 1. Clarify | `/grill-me` | Open questions resolved |
| 2. Write the PRD | `/spec-writer` | `docs/prd/NNN-slug.md` from `TEMPLATE.md` |
| 3. Adversarial gate | `/grill-me` **on the PRD** | Contradictions, gaps, scope creep |
| 4. Slice to backlog | `/slice-the-spec` | Vertical slices, dependency-ordered |
| 5. Split if still big | `/story-splitting` | Sprint-sized tickets |

Stages 1–5 run **per epic**. Stage 0 runs once over the whole idea.

**Step 3 is a gate, not a formality.** Point `/grill-me` at the written document,
not at the idea — the point is to find what the *spec* got wrong. Set
`grilled: true` and move `status` to `approved` only once nothing critical is
outstanding. Anything unresolved goes in § 9 Open Questions and the PRD stays
`draft`.

`TEMPLATE.md` mirrors the `spec-writer` skill's 10-section structure, so the
skill's output drops straight in without reshaping.

## Writing rules

- **Business language, not implementation.** "A user can cancel up to 2h before
  the slot" — not "add a `cancelled_at` column".
- **Every requirement testable.** If you cannot write a pass/fail check for it,
  it is a goal, not a requirement — put it under Goals.
- **State non-goals explicitly.** Most scope disputes are about what was assumed
  in, not what was written in.
- **Open questions stay open.** Do not resolve them by guessing; leave them
  listed until answered, and mark the PRD `draft` while any blocker remains.
