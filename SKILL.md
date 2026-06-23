
# SKILL.md — Capacity Planner

## Current phase
**Phase 1 — prototype built (2026-06-10).** Spec fully implemented in `index.html`.
Math reconciles to the hand-checked sample. Not yet in git; not yet shared.

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
platform. See CLAUDE.md for the full rejected list.

## Phase 1c — UI redesign + public-prep (2026-06-10)
Rebuilt `index.html` for intuitiveness: hero thesis ("More AI agents ≠ fewer humans" +
the volume-game→AHT-game reframe), a live results strip (humans needed / blended cost /
weekly cost), grouped inputs with tooltips (no per-lever paragraphs), two captioned charts
(Reality vs Ignoring-difficulty + a clean floor line). Cut the planning lens. Naming fixed:
humans = "humans", bot = "AI agents" (no more agent collision). `derive()` math untouched
(still 44 / $4.46 on defaults). Removed the two root docs (`wfm-capacity-the full screen +
model.md`, `wfm-capacity-planner-HANDOFF.md`) — model now lives in the skill references +
README. Added MIT `LICENSE`. README rewritten from `~/Downloads/capacity.md` (builder voice,
2026 Gartner/Robert Half citations, the model formula block). "AI-first" buzzword scrubbed.
`CLAUDE.md` gitignored (internal). Repo ready to flip public + GitHub Pages.

## TODO — induced-demand lever (queued 2026-06-10, not started)
Add a SECOND lever: induced demand. Source/justification now cited in README — the
Forbes/Assembled "Deflection Illusion" passage (Ryan Wang, Apr 2 2026): turning on AI
*creates* contacts that never existed before (suppressed easy questions; the barrier to
asking drops), so `volume` is not fixed — it rises with coverage. This is the demand-side
twin of the existing rising-residual-AHT mechanism (survivors are harder vs. there are
more-and-easier of them). Both push the headcount floor UP — gives the hero a second leg.

Build plan (full design worked out in the session that queued this):
1. **Math.** `V(coverage) = V₀ × (1 + ε·coverage)`. New input `ε` = "induced demand at
   full AI" (e.g. 0.3 = +30% volume at 100% coverage). Everything downstream already keys
   off `volume`, so it recomputes for free.
2. **Toggle, default OFF / ε=0.** With it off the screen is byte-for-byte today's artifact
   — must still reconcile to 31 vs 44, $4.46, $39.2k. Adding a second mode, not changing
   the existing one.
3. **DECISION (mine to make at build): induced contacts are EASY, not average mix.** They
   pile on the easy end; AI eats most; only `(1−success)` reach a human at easy AHT. Net:
   total contacts up a lot, AI cost up, human headcount up modestly, residual AHT drifts
   *down* slightly (mix got easier). That tension is the interesting moment. (Trivial
   alternative = uniform `V` scaling; weaker claim, don't use unless asked.)
4. **Readout + honest flag.** Show "contacts created: +N/wk" under the levers. Floor
   visibly rises when toggled on. CRITICAL: with ε>0, blended cost = cost(a)/V(a) and the
   concavity / no-sweet-spot proof NO LONGER HOLDS — an interior cost minimum becomes
   possible. Regime badge needs a state: "induced demand on — clean-model no-sweet-spot
   guarantee suspended." This is the project showing the exact boundary of its own proof.
   Update CLAUDE.md "Hard rules" / caveats accordingly (induced demand is one of the
   already-listed real-world mechanisms that reintroduce interior optima).
5. **Re-verify.** ε=0 must reproduce the sample unchanged; one ε>0 value to sanity-check
   the new path. Re-check against `skills/ai-capacity-planner/references/the_model.md`.

## Phase 2 — article-gap features + walkthrough refactor (2026-06-23)
Implemented all three features from `FUTURE-FEATURES.md` (prompted by Fernando Duarte's
article), then refactored the UI into a sequential 5-stage walkthrough.

### Features added (all off by default, reconcile to sample at defaults)
- **Feature A — AI supervision:** `supMin` input (human oversight min per AI-resolved contact).
  Adds `supHours = resolutions × supMin / 60` to `humanHours`. Raises floor, widens naive gap.
- **Feature B — Growth reframe:** `growth` input + Snapshot/Growth toggle. Second `derive()` call
  with scaled volume. Reframes KPIs: "serve +X% volume with Y humans vs Z at 0% AI."
- **Feature C — Assisted work (multiplier):** `assistShare` + `assistFactor` inputs. Multiplier
  approach (keeps two AHT endpoints): `assistMix = 1 - assistShare × (1 - assistFactor)`.
  Lowers human hours without removing tickets from the queue.

### Walkthrough refactor
Replaced the 12-input single screen with a 5-stage stepper:
1. **Staffing** — base model inputs (volume, AI coverage, success, AHT, cost)
2. **Oversight** — supervision minutes per AI resolution
3. **Assisted work** — AI pre-processing speedup (channel-agnostic framing)
4. **Growth** — volume growth projection + Snapshot/Growth toggle
5. **Summary** — read-only view of all inputs + final KPIs

Each stage reveals its own inputs. Previous stages collapse to a summary chip (clickable
to navigate back). Stage-specific insight callouts explain the key takeaway. `derive()`
and `readInputs()` untouched — stage management is pure visibility/CSS.
