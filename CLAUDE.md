# Rally IQ v2 — AI Badminton Coach

Upload a badminton video → per-shot detection, classification and
coaching. Built from scratch; v1 at `~/Documents/Github/Rally IQ` is
**reference only** — never import from it or edit it.

## Docs

- `docs/plan.md` — the one design doc (goals, v1 lessons, scope, phases, curriculum).
- `docs/status.md` — what's done and what's next. Update it as pointers move.
- `learn/progress.md` — learning state (owned by the `study-coach` agent).
- Keep docs short. No LLD. Phases are detailed only when picked up.

## Ways of working

- **One pointer at a time.** Each pointer = one PR with its expected output
  written in the plan *before* implementation. Finish, stop, explain, and
  wait for confirmation before the next.
- **Teacher mode.** Explain concepts, libraries and algorithms in plain
  language — the *why* — using backend/AWS analogies where honest.
- **End-to-end first, measure always.** Accuracy changes are scored against
  the hand-labelled ground truth via the eval harness.
- **Learning rules for AI use:**
  - Learning toys (`learn/`): Rahul writes them; Claude hints and reviews only.
  - Product code: Claude may draft, but it doesn't merge until Rahul can
    explain it back. If he can't, it becomes a learning task first.
  - Don't introduce a concept into product code before its learning module
    (plan §5) has at least started.

## Code conventions

- Python 3.11, `uv`, `pytest`, `ruff`. Tests run green on a fresh clone and in CI.
- Clarity over cleverness. Precise, minimal comments that state intent.
- CV/ML code: name preprocessing steps as functions; comment array shape,
  dtype, units and colour space at every transformation.
- Typed dataclasses between stages — no raw dicts.
- One source of truth per signal. Physical units. `None` for missing,
  never `0.0`.
- All tunables in one config module.
- No secrets, videos or model weights in git.
- **No AI attribution in git.** Commits and PRs carry no `Co-Authored-By:
  Claude` lines, no "Generated with Claude Code" footers, no AI mentions.
  Rahul is the sole author. This repo goes on his resume.
- LLM prompts versioned in files; the LLM phrases measured numbers and never
  invents metrics.
