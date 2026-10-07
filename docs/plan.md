# Rally IQ v2 — Founding Plan

> The one design doc for v2. Kept short on purpose: v1 produced ~10k lines
> of docs for ~12k lines of code and never ran end to end. Phases are
> detailed **one at a time**, when picked up — not all up front.
>
> Status lives in `docs/status.md`. Learning progress lives in
> `learn/progress.md`.

---

## 1. Goals

1. **A working production product** — upload a badminton video, get
   per-shot coaching you can trust.
2. **Understand every concept the product uses** — no black boxes. If we
   can't explain it, we haven't finished it.
3. **Get competent at AI tools on a real problem** — and build a
   portfolio (measured results, write-ups, a deployed URL) that helps land
   a job in AI.

## 2. Lessons from v1 (`~/Documents/Github/Rally IQ`, reference only)

| What happened in v1 | Rule for v2 |
|---|---|
| Months spent perfecting identity/tracking + replay tooling; shot classification, shuttle, LLM, API, DB, frontend never built. | **End-to-end first.** A crude full pipeline before any stage is polished. |
| No ground truth — quality argued from debug videos; impossible numbers (16 m/s ankles, 559 "swings" on a 9½-min clip) went unnoticed. | **Label first, measure always.** Every accuracy change is scored against hand labels. |
| Hardest case (amateur doubles + bystanders) used as the main benchmark. | **Easy case first** (singles, static camera), hard cases added and measured later. |
| Velocities in full-frame pixels; same signal computed three ways; 0.0 stored for "missing". | **One source of truth per signal**, physical units, `None` for missing. Sanity checks on outputs. |
| AI wrote most code; concepts weren't understood, so bugs weren't recognisable. | **Learn → toy → tool → measure → explain back** for every concept (§5). |
| Tests listed in docs but zero test files existed; env couldn't run. | Reproducible env + CI from day one. Tests must run on a fresh clone. |
| Docs larger than code; LLD 3,191 lines. | One plan doc + one status board + short learning notes. |

**v1 concepts worth keeping:** user-confirmed anchor then tracking (not
per-frame matching); explicit tracking state machine
(locked / lost / re-acquire); swappable pose backend behind an interface;
thresholds scaled by body size; structured "why" behind every confidence
score; "gap > wrong" — skip rather than guess.

## 3. Scope of the first working slice

- **Singles**, **static camera**, our own / chosen recordings.
- **Target selection:** user taps themselves in one frame → tracker locks
  on. No onboarding photos, no ReID (added in Phase 2, *measured against*
  this baseline).
- **Shot types:** serve, clear, smash, drop, net shot (+ "other" /
  not-a-shot).
- **Output:** per-shot list (time, type, confidence, key measurements)
  + 2–3 coaching cues phrased by Claude from measured numbers only.
- **Out of scope for now:** doubles, moving camera, racket tracking,
  court homography, tactical analysis.

## 4. Phases and time budget

Budget: **10 h/week** → roughly **4 h learning + toy**, **5 h building**,
**1 h write-up + review**. Week counts are estimates; we re-plan at the
end of each phase.

| Phase | Learning modules (§5) | Product outcome | Est. weeks |
|---|---|---|---|
| 0 — Foundation | M1, M2 | Repo + CI, video I/O, labelled ground truth, eval skeleton | 3–4 |
| 1 — Thin end-to-end | M4, M5, M7, M8, M11, M12 | Detect → track (tap-to-lock) → pose → swing candidates → simple shot classifier → Claude coaching → HTML report, all scored by eval | 8–10 |
| 2 — Real identification | M3, M6 | ReID + doubles, measured vs Phase 1 tap-to-lock | 4–5 |
| 3 — Shuttle + trained classifier | M9, M10 | Shuttle tracking; learned shot classifier; contact timing | 6–8 |
| 4 — Production + AI features | M13, M-D | API + queue + Postgres + object store + small frontend, deployed; coach-chat agent | 6–8 |

Total ≈ 6–8 months at 10 h/week.

## 5. Learning curriculum

**Method for every module:**
1. **Learn** — the concept and why Rally IQ needs it.
2. **Toy** — a small from-scratch version *we write* (Claude tutors/reviews, doesn't author).
3. **Tool** — use the production library, now understood.
4. **Measure** — on our labelled footage.
5. **Explain back** — short note in `learn/notes/`; no merge until we can explain it.

| # | Module | Toy exercise | Phase |
|---|---|---|---|
| M1 | Images & video as arrays (numpy, BGR/HSV, fps, OpenCV I/O) | Frame quality scorer (brightness, sharpness) + the labelling tool | 0 |
| M2 | Classic CV (thresholds, Canny, Hough, morphology) | Detect the net tape; document why it's fragile | 0 |
| M3 | Neural-net basics (tensors, conv, PyTorch training loop) | Small CNN: player crop vs not | 2 |
| M4 | Object detection (boxes, IoU, NMS, confidence, mAP; YOLO) | Write IoU + NMS; measure YOLO on labelled frames | 1 |
| M5 | Multi-object tracking (Kalman, Hungarian, SORT → BoT-SORT) | Minimal SORT (~150 lines); count ID switches | 1 |
| M6 | Re-identification & embeddings (cosine sim, triplet loss, galleries, ROC thresholds) | Similarity matrix of you / partner / opponent; pick a threshold from ROC | 2 |
| M7 | Pose estimation (keypoints, heatmaps, top-down, COCO-17 vs MP-33, 2D limits) | MediaPipe vs ViTPose vs hand-clicked keypoints | 1 |
| M8 | Signals over time (smoothing, velocity, normalisation, peaks) | Reproduce v1's 16 m/s ankle bug, then fix it | 1 |
| M9 | Small fast-object tracking (TrackNet heatmaps) | Shuttle on our clips; measure detection rate | 3 |
| M10 | Classification (features + XGBoost vs sequence models; splits, leakage, imbalance) | Shot classifier; split by video, not frame | 3 |
| M11 | Evaluation (event matching with tolerance, precision/recall, confusion matrix) | The eval harness itself | 0–1 |
| M12 | LLM engineering (structured output, grounding, prompt versions, LLM evals) | Coaching phraser that never invents numbers + its eval | 1 |
| M13 | Agents & tool use | Coach chat over session data | 4 |
| M-D | Production ML (queues, GPU inference, model versioning, deploy, observability) | Deployed pipeline | 4 |

**Core papers to read (with Claude as reading partner):** SORT, BoT-SORT,
OSNet, ViTPose, TrackNet (v2/v3).

## 6. Phase 0 — pointers (expected output defined up front)

Each pointer = one PR. Detailed further on pickup.

| # | Pointer | Expected output ("done when…") |
|---|---|---|
| 0.1 | Repo + environment | `uv` project, Python 3.11 pinned, `pytest` + `ruff`, GitHub Actions CI. Fresh clone → `uv sync && uv run pytest` green; CI green. |
| 0.2 | Data layout | Videos/weights **not** in git; `data/` gitignored with a manifest (`data/manifest.json`: path, fps, frames, duration, format). Singles clips copied from v1 fixtures. |
| 0.3 | Video I/O (M1) | `rallyiq/video.py`: open, metadata, iterate, stride-sample, clear error on bad file. Tests on a tiny generated video. |
| 0.4 | Labelling tool + ground truth (M1) | Our own OpenCV keyboard labeller → `labels/<video>.json` (contact frame, shot type, near/far). Schema documented. **≥ 5 singles videos labelled.** |
| 0.5 | Classic CV exploration (M2) | Learning-only notebook: net-tape detection attempt + note on failure modes. No product code. |
| 0.6 | Eval skeleton (M11) | Event matching with ±N-frame tolerance, per-type precision/recall, confusion matrix. Tested on synthetic data. A trivial baseline predictor scored against real labels = our floor. |

Phases 1–4 are broken into pointers when picked up.
