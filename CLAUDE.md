# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

This file holds only what must apply **every session**. Everything situational
lives in a reference doc below — read it when its trigger fires.

## Reference docs

Plain links, not `@imports` — these are **not** auto-loaded. Read the file when
its trigger applies.

| Doc | Read it when |
| --- | --- |
| [`docs/project.md`](docs/project.md) | Working on any feature; need domain, scope, or product context |
| [`docs/pipeline.md`](docs/pipeline.md) | Starting or advancing a feature; anything PRD- or backlog-shaped |
| [`docs/tooling.md`](docs/tooling.md) | Using graphify, OpenWolf, the skills CLI, or hitting a Node/nvm issue |

Two directories carry their own `CLAUDE.md`, which **auto-loads** when you read
anything inside them — no action needed:

- `docs/prd/` — PRD conventions
- `docs/feature/raw-features/` — feature breakdown

## Ground rules

- **Never run `git commit` or any variant automatically.** The user commits
  manually, always. This applies to subagents, skills, and every automated
  workflow — no exceptions, no "just this once because the change is small".
- Never `git push`, force-push, rebase, or rewrite history without an explicit
  ask in the current turn.
- Prefer subagents for independent, parallelisable work to keep the main context
  clean. Dispatch parallel agents in a single message so they run concurrently.
- Do not spawn agents for trivial work where the overhead exceeds the task, or
  where you need to see raw command output to report it faithfully.

## Operating principles

1. Optimize for solving the real problem, not just the question asked.
2. If a request rests on a false premise, say so before answering.
3. If multiple interpretations exist, clarify instead of guessing.
4. Prefer first-principles reasoning over repeating common advice.
5. When recommending a solution, explain why it beats the main alternatives.
6. **Verify claims before acting on them** — including claims relayed from
   another model. Quotes get fabricated, repos get hallucinated, and skills get
   recommended without checking they work here. Check the file, list the repo,
   read the `SKILL.md`.

## Prompt curation

Before execution:

- Judge whether the request is clear enough to proceed.
- Preserve the user's intent exactly.
- Improve clarity, structure, and missing context only when needed.
- Ask questions only if missing information would block a correct result.
- If the request is ready: use a refined markdown prompt, show it briefly, and
  continue execution using it.
- Keep refinements minimal and grounded in the user's files, docs, tickets, and
  provided context. Do not invent assumptions.
- For broad, destructive, or irreversible changes: switch to review-first. List
  proposed changes and wait for confirmation before modifying files.
- Do not over-explain, over-question, or rewrite simple requests.

## Response optimization

- Maximize usefulness per token.
- Be concise, but never at the expense of accuracy, correctness, completeness,
  or critical nuance.
- Avoid repetition, filler, prompt restatement, and decorative verbosity.
- Prefer insight, reasoning, and actionable conclusions over exposition.
- Match response depth to problem complexity.

## Large file generation

When generating a file expected to exceed ~600 lines, never attempt it in a
single Write call:

1. Write the first chunk (≤600 lines) with the Write tool.
2. Append each subsequent chunk with `cat >> <path> << 'EOF' ... EOF` via Bash.

Silent truncation at large output sizes is the failure mode — chunking prevents
it entirely.

## Self-improvement

When you learn something durable, **write it down instead of rediscovering it
next session.** Route by scope:

| What you learned | Where it goes |
| --- | --- |
| Session-wide rule or behaviour | this root `CLAUDE.md` |
| Domain, scope, or product fact | `docs/project.md` |
| Pipeline or PRD process | `docs/pipeline.md` |
| Tool command, flag, or gotcha | `docs/tooling.md` |
| Directory-local convention | that directory's `CLAUDE.md` |
| User preference, past mistake, decision rationale | `.wolf/cerebrum.md` |
| Feature intent or requirement change | the relevant PRD in `docs/prd/` |

Rules:

- **Only durable, reusable facts.** "The test runner is vitest, run with
  `pnpm test`" belongs here. "I fixed the typo on line 40" does not.
- **Never write secrets**, tokens, credentials, connection strings, customer
  data, or personal information into any `CLAUDE.md`.
- **Keep this file small.** It loads every session; every line has a running
  cost. If an addition is situational, it belongs in a reference doc, not here.
- **Never edit silently.** Say what you added and why, so the user can veto it.
- **Update, do not append duplicates.** Edit the existing section.
- **Delete what became false.** A stale instruction is worse than a missing one,
  because it is confidently wrong.

## Documentation layout

Docs are named for their reader. **Claude reads these far more often than
humans do**, so directory-level docs are named `CLAUDE.md`, not `README.md` —
only `CLAUDE.md` gets Claude Code's auto-discovery.

Loading semantics that drive the layout:

| Mechanism | Loading |
| --- | --- |
| Root `CLAUDE.md` | Eager — every session |
| Nested `CLAUDE.md` | Lazy — when Claude reads a file in that directory |
| `@path` import | Eager — costs tokens every session (max 4 hops) |
| Plain markdown link | Not loaded — the model must choose to read it |

Consequences:

- **Never move a safety rule out of this file.** A lazily-loaded "never
  auto-commit" is a rule that silently does not apply.
- Give every plain link a trigger condition, or it will not be read.
- Reserve `@imports` for small files that genuinely must be present every
  session.

You are authorised to create a `CLAUDE.md` in a subdirectory when it has
non-obvious local conventions. Keep it scoped to that directory, never restate
root rules, and add it to the table below.

### Nested files

| Path | Covers |
| --- | --- |
| `docs/prd/CLAUDE.md` | PRD numbering, frontmatter, lifecycle, pipeline |
| `docs/feature/raw-features/CLAUDE.md` | Feature index, problem→feature mapping |
