# HANDOFF — AI-First Capacity Planner (2026-06-10)

**Read this to resume. You are picking up a design conversation, mid-stream, right before building.**
Builder: solo, ships fast, plain JS, **no TypeScript ever**, no build ceremony. This is an
interview proof-of-work artifact for a WFM / CX capacity-planning context, not a startup.

---

## What we're building

A single-screen **AI-first capacity planning** prototype. Sweep one lever (**AI coverage**)
and watch two outputs move — **headcount** and **blended cost per contact** — plus a
**planning lens** that interprets *why* the leftover work is slow.

It is the **union of three things**, and the union is the whole point:
- **A published industry staffing model** — staffing mechanics (residual volume → headcount). Stops at "how many."
- **A blended-cost lens** — blended AI-vs-human cost per contact. Asks "is it worth it."
- **The builder's own mechanism** — residual AHT *rises* as AI takes more, because AI eats
  the easy tickets first and the hard ones stay. This is what changes the headcount/cost
  mechanics (it inflates the human floor; it does **not** bend cost into a sweet spot).

Source of truth for the math: see `wfm-capacity-the full screen + model.md` (the full
ASCII screen + model). This file is self-contained enough to resume without it, but it's
adjacent if you want it.

---

## The model (revised 2026-06-10 — supersedes the old "locked" block)

The first "locked" model overclaimed. Tightened version (full derivation + screen
in `wfm-capacity-the full screen + model.md`):

```
a (effective automation) = coverage × success
human tickets            = volume × (1 − a)

aht(p)        = Easy + (Hard − Easy)·p        ← linear difficulty, p in [0,1], AI eats easy first
baseline AHT  = (Easy + Hard) / 2             ← all-ticket mean; the naive math uses this
residual AHT  = Easy + (Hard − Easy)·(1 + a)/2  ← closed-form avg over surviving slice [a,1]

human hours   = human tickets × residual AHT / 60
headcount     = human hours / productive hrs per agent

attempts      = volume × coverage
resolutions   = volume × a
AI cost       = attempts×fee  (per-attempt mode)  OR  resolutions×fee  (per-resolution mode)
                — pick ONE mode; the other fee is zero. Leading AI agents bill per-resolution.
human cost    = human hours × human $/hr
blended $/contact       = (AI cost + human cost) / volume
fully-loaded $/agent/wk = human $/hr × productive hrs per agent   ← display next to headcount
```

Inputs (levers): weekly volume, AI coverage %, AI success %, **easy-ticket AHT**,
**hard-ticket AHT**, productive hrs/agent/wk, human $/hr, **AI billing mode**
(per attempt | per resolution), AI fee.

**Key results that changed the story:**
- **Residual AHT is now a visible closed form** with an explicit difficulty-shape
  assumption (straight-line ramp, easy tickets first). Two endpoints alone don't
  determine it — the shape does.
- **No cost sweet spot exists — within this simplified continuous model.** Blended
  cost is *concave* in coverage for any increasing difficulty curve
  (`d²/da² = −(W/60)·aht′(a) ≤ 0`), so an interior *minimum* is impossible. Cost can
  only fall, rise, or rise-then-fall. The marker shows the regime: "cheapest at max"
  / "cheapest at zero" / "worst at X%." **Caveat to state up front:** the proof holds
  under the stated assumptions (continuous coverage, smooth difficulty, linear
  per-unit costs). Real WFM can reintroduce interior optima via stepwise staffing,
  minimum shift coverage, vendor tiers/volume discounts, fixed platform fees, or SLA
  penalties. Frame the claim as "in the clean model," not as a universal law.
- **The headcount floor is the hero**, set by `(1 − success)` and made costlier
  than naive math expects by the rising residual AHT. Robust across normal
  imperfect-AI / mixed-difficulty settings (it degenerates only at the edges:
  success = 100% removes the floor; easy AHT = hard AHT removes the naive gap).

---

## Decisions locked

1. **Build that *kind* of staffing model — but the union, not a copy.** Copying a plain
   headcount calculator is not unique. The cost lens + the rising-AHT mechanism + the diagnosis
   panel are what make it defensible and not a book report.
2. **Two AHT inputs, not one** (easy-ticket AHT + hard-ticket AHT). The human average drifts
   toward the hard number as coverage rises, on its own. This is mechanically honest.
3. **Blended cost is emergent, not preordained — and has NO interior optimum.**
   (Revised.) The old claim "cost turns up to a cost-optimal coverage" is
   mathematically false: cost is concave in coverage, so it's monotone or
   worst-in-the-middle, never best-in-the-middle. Don't draw a sweet spot. The
   falsifiable dare becomes *"there is no automation sweet spot on cost."*
4. **Planning lens reframes, doesn't compute** (was "diagnosis panel"). Same AHT
   number → three actions: *structural* (accept, staff for it) · *capability*
   (train, don't add heads) · *load* (fix concurrency first). Framed explicitly as
   an interpretation lens — no input proves which one — not a computed diagnosis.
   This is the builder's original contribution; neither the staffing model nor the cost lens has it.
5. **Frame it as a planning model / simulator, not a forecast.** Inputs are scenario
   assumptions, stated up front. Anchor at least one input to a real published number
   (39% reach live agents · industry forecast ~$3/resolution by 2030 · 73% ACW flat-or-up) so it
   can't be dismissed as made up.

## Decisions REJECTED (don't reopen these)

- ❌ **A fuzzy "how fast does work get harder" difficulty slider.** Over-engineering. Replaced
  by the two-AHT-endpoints approach.
- ❌ **A single flat AHT number.** Too simple — survivors must get harder for the
  headcount-floor cost story to work. (Note: even the two-AHT model can yield a
  monotone-down cost curve — that's now an accepted, honest regime, not a failure.
  The "moment" lives in the headcount floor + the no-sweet-spot proof, not in a
  cost turn-up.)
- ❌ **A cost-optimal-coverage marker / U-shaped cost curve.** (Added 2026-06-10.)
  Proven impossible: blended cost is concave in coverage. Never draw an interior
  minimum.
- ❌ **Building a full WFM platform** (scheduling, adherence, rostering, intent-level breakdowns,
  saved scenarios). Scope creep. One screen, one lever, two curves, one diagnosis strip.
- ❌ **Leading with the thesis / trend recap.** The framing is consensus as of 2026; the
  interviewer read the same articles. Lead with the model and the counterintuitive result.

## The three "moments" the screen should create (revised)

1. **Headcount drops to a floor, not zero — and the floor costs more than naive
   math says.** The HERO. Floor is set by `(1 − success)`; rising residual AHT
   makes it pricier than expected. Robust across normal imperfect-AI /
   mixed-difficulty settings (degenerates only at the edges: success = 100% or
   easy AHT = hard AHT). Kills "AI does 80%, cut 80% of staff."
2. **No cost sweet spot — in the clean model.** Blended cost is provably monotone
   or worst-in-the-middle, never best-in-the-middle, *under the stated assumptions*
   (continuous coverage, smooth difficulty, linear costs). The marker names the
   regime. The falsifiable dare: hand over the sliders, *"find me the optimum —
   there isn't one in this model."* (Name the real-world caveats — stepwise
   staffing, vendor tiers, fixed fees, SLA penalties — before they do.)
3. **Naive-vs-model headcount counter** — the old volume÷baseline-AHT math
   under-staffs (sample: 31 vs 44, a 13-person gap). Show the gap as a number.

---

## OPEN QUESTIONS — RESOLVED 2026-06-10

1. Layout holds — one lever, two curves, planning-lens strip. See the updated
   ASCII in `wfm-capacity-the full screen + model.md`.
2. **Hero = headcount floor** (not cost). Cost is the honest second chart, framed
   as "no sweet spot," not as a turn-up. Decided after proving the cost curve has
   no interior minimum.
3. **AI billing = one-mode toggle** (per attempt OR per resolution; other fee = 0).
   Mirrors common per-resolution AI pricing.
4. **Human cost input stays $/hr**, with fully-loaded $/agent/wk displayed alongside
   headcount (the number a staffing manager actually budgets with).

## Build defaults when greenlit

Plain HTML + JS, single file, no framework, no TypeScript, no build step. Two line charts.
Sliders/number inputs on the left, charts on the right, diagnosis strip across the bottom.
