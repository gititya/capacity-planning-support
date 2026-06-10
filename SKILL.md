# SKILL.md — AI-First Capacity Planner

## Current phase
**Phase 1 — prototype built (2026-06-10).** Spec fully implemented in `index.html`.
Math reconciles to the model doc's hand-checked sample. Not yet in git; not yet shared.

**Phase 1b — packaged & pushed (2026-06-10, later same day).** Now a git repo, pushed
to **github.com/gititya/capacity-planning-support** (PRIVATE). All external-party names
(Opus / Intercom / Fin / Figma / Gartner) scrubbed from committed files; the Opus
transcript `original strategy assessment.md` is gitignored (local only, NOT in repo —
keep it that way). Added:
- `skills/ai-capacity-planner/` — the model as a deterministic, stdlib-only agent skill
  (SKILL.md + `scripts/ai_capacity_modeler.py` + two references). Verified it reproduces
  the hand-checked numbers and flips through all three cost regimes. Positioned as the
  AI-first layer that runs BEFORE Erlang-C (hands residual off to `capacity-planner`).
- `README.md` — written in the builder's voice, references github.com/gititya/MARS.
Ecosystem research done: only one comparable skill exists in the wild
(`alirezarezvani/claude-skills` business-operations `capacity-planner`, classical Erlang-C,
no AI mechanics) — the AI-first delta is unoccupied whitespace. Hypothesis confirmed.

## What exists
- `index.html` — the working prototype (single file, plain JS, Chart.js via CDN).
- Two reference docs (model, handoff).

## How to run
Open `index.html` in any browser. No server, no build, no install. (Needs internet for
the Chart.js CDN scripts on first load.)

## Built (against the spec)
- Levers panel (left): volume, coverage [swept], success, easy/hard AHT, productive hrs,
  human $/hr (+ fully-loaded $/agent/wk), per-attempt|per-resolution radio, AI fee.
- Hero headcount chart (top): model curve + faded naive curve, red dashed floor line
  "(1 − success)", live coverage marker + dot.
- Blended-cost chart (below): live **regime badge** (cheapest-at-max / cheapest-at-zero /
  worst-at-X%), worst-point marker when that regime hits.
- Naive-vs-model counter + automation rate + model staffing $/wk under the levers.
- Planning-lens strip (bottom): structural / capability / load — interpretation only, computes nothing.

## Next steps (when picked up)
1. **Decide git/GitHub** — recommended: init private now, flip public + GitHub Pages when
   ready to share with the interviewer (zero secrets, so public is safe).
2. Optional: pin SRI `integrity` hashes on the two CDN `<script>` tags.
3. Optional polish: eyeball layout in a browser; tune chart spacing/labels.
4. Optional: a short README framing the artifact for the interviewer (the model, the
   counterintuitive result, the falsifiable dare).

## Demo script (for the interview)
- Lead with the model + the counterintuitive result, **not** the trend recap.
- Show moment 1 (headcount floor) → moment 3 (naive-vs-model gap) → moment 2 (no sweet spot).
- The dare: hand over the sliders. Bump AI fee high in per-attempt mode to flip the regime
  badge through all three states — "find me the optimum; there isn't one in this model."
- Name the real-world caveats (stepwise staffing, vendor tiers, fixed fees, SLA penalties)
  before they do.

## Do not reopen
Cost sweet-spot marker (proven impossible), difficulty slider, flat single AHT, full WFM
platform. See CLAUDE.md + HANDOFF for the full rejected list.
