# Tooling

> Referenced from root `CLAUDE.md`. graphify, OpenWolf, skills, and runtime.

## graphify

This project has a **single root-level** knowledge graph at `graphify-out/`.
(Ignore any guidance written for multi-project layouts — there are no
subprojects here.)

The graph does not exist until there is code to index. Build it with
`graphify update .` from the repo root; `graphify watch .` keeps it live.

Once `graphify-out/graph.json` exists, prefer graph tools over raw file scanning
for codebase questions — they return a scoped subgraph, usually far smaller than
grep output:

- `graphify explain "<concept>"` — focused explanation of a node and neighbours.
- `graphify path "<A>" "<B>"` — how two things relate.
- `graphify-out/wiki/index.md` — broad navigation, if present.
- `graphify-out/GRAPH_REPORT.md` — architecture review only, when the above is
  not enough. It is large; do not open it casually.

After modifying code, run `graphify update .` to keep the graph current
(AST-only, no API cost). Fall back to Grep/Glob/Read when the graph genuinely
does not cover what you need.

## OpenWolf

OpenWolf (`.wolf/`) manages cross-session context. `.wolf/OPENWOLF.md` holds the
full protocol; the essentials:

- Check `.wolf/anatomy.md` before reading a file — if its description suffices,
  do not read the file.
- Check `.wolf/cerebrum.md` before generating code, and respect every entry,
  especially `## Do-Not-Repeat`.
- Update `.wolf/cerebrum.md` when you learn a preference, convention, gotcha, or
  decision. The bar is deliberately low.
- Update `.wolf/anatomy.md` after creating, deleting, or renaming files.
- Log bugs to `.wolf/buglog.json` per the protocol.

Useful commands: `openwolf status`, `openwolf scan`, `openwolf dashboard`.

The pm2 daemon is not installed, so background daemon features are inactive.
Everything above works without it.

## Skills

Installed project-level in `.claude/skills/` (copied, not symlinked, so they are
self-contained and reviewable). Managed with the `skills` CLI:

| Skill | Stage | Invocable by | Source |
| --- | --- | --- | --- |
| `slice` | 0 — epic split | user only | `thoughtbot/rails-consultant` |
| `grill-me` | 1 & 3 — interview / gate | user only | `mattpocock/skills` |
| `grilling` | implementation behind `grill-me` | model | `mattpocock/skills` |
| `spec-writer` | 2 — PRD | model | `kambleakash0/agent-skills` |
| `slice-the-spec` | 4 — PRD → backlog | model | `kambleakash0/agent-skills` |
| `story-splitting` | 5 — split oversized stories | model | `eferro/augmented-lean-delivery` |
| `hamburger-method` | 5 alt — layered slicing, no obvious split | model | `eferro/augmented-lean-delivery` |

**Deliberately not installed:** `duc01226/easyplatform --skill story`. It is
SPIDR-based and looks right, but hard-references 10 repo-local files
(`docs/project-reference/*`, `docs/project-config.json`, …) that do not exist
here. It would fail or fabricate. `story-splitting` covers the same need
portably.

- Add: `npx skills add <owner>/<repo> -s <skill> -y --copy`
  (repeat `-s` per skill; comma-separated lists do **not** work).
- Inspect before installing: `npx skills add <owner>/<repo> -l`.
- Update: `npx skills update -p`. List: `npx skills list`.
- Skills run with full agent permissions. Read a `SKILL.md` before trusting it,
  and note that some skills are thin shims that delegate to another skill —
  install the dependency too or the chain breaks silently.
- `skills-lock.json` pins each skill's source and content hash. Commit it.
- Before adding a skill from a large monorepo-style pack, check its `SKILL.md`
  for references to repo-local paths. Skills written for one codebase often
  assume that codebase's docs exist.
- The CLI writes **two copies**: `.claude/skills/` (what Claude reads) and
  `.agents/skills/` (agent-agnostic mirror for Cursor/Copilot/etc). They are
  independent copies, so never hand-edit one — change skills through the CLI, or
  apply the same edit to both.

## Node runtime

Pinned to **24** via `.nvmrc`. Switch with `nvm use` (or `nvm use 24`).

- `nvm` is a shell function, not a binary — it is unavailable in
  non-interactive shells. To use it inside a scripted command:
  `export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"; nvm use 24`
- Shell state does not persist between tool calls; source it each time.
- The `skills` CLI requires Node >= 22.20.0. On 22.12.0 it emits `EBADENGINE`
  and still works; on 24 the warning is gone.
