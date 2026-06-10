# CLAUDE.md — AI-First Capacity Planner

## What this is
A single-screen **AI-first capacity planning** prototype. An interview proof-of-work
artifact for a WFM / CX capacity-planning context — not a startup, not a
product. Sweep one lever (**AI coverage**) and watch two outputs move — **headcount**
and **blended cost per contact** — plus a **planning lens** that interprets *why* the
leftover work is slow.

It is the **union of three things**, and the union is the point:
- **A published industry staffing model** — staffing mechanics (residual volume → headcount). Stops at "how many."
- **A blended-cost lens** — blended AI-vs-human cost per contact. Asks "is it worth it."
- **The builder's mechanism** — residual AHT *rises* as AI takes more, because AI eats
  the easy tickets first and the hard ones survive. This inflates the human floor.

## Stack / constraints
- **Single file: `index.html`.** Plain HTML + JS. No framework, no build step, **no TypeScript**.
- Charts: **Chart.js v4 + chartjs-plugin-annotation**, loaded from jsDelivr CDN at runtime.
- No backend, no API keys, no secrets, no Firebase. Open the file in a browser — that's it.

## The model (source of truth: `wfm-capacity-the full screen + model.md`)
```
a (effective automation) = coverage × success
human tickets            = volume × (1 − a)
baseline AHT             = (Easy + Hard) / 2                    ← naive math uses this
residual AHT             = Easy + (Hard − Easy) · (1 + a) / 2   ← avg over surviving slice [a,1]
human hours              = human tickets × residual AHT / 60
headcount                = human hours / productive hrs per agent
AI cost (one mode only)  = attempts×fee (per-attempt)  OR  resolutions×fee (per-resolution)
human cost               = human hours × human $/hr
blended $/contact        = (AI cost + human cost) / volume
fully-loaded $/agent/wk  = human $/hr × productive hrs per agent
```

## The three "moments" the screen creates
1. **Headcount drops to a floor, not zero — and the floor costs more than naive math says.**
   The HERO. Floor set by `(1 − success)`; rising residual AHT makes it pricier than expected.
2. **No cost sweet spot — in the clean model.** Blended cost is provably *concave* in coverage
   (`d²/da² = −(W/60)·aht′(a) ≤ 0`), so it's monotone or worst-in-the-middle, **never**
   best-in-the-middle. The regime badge names which regime the inputs land in.
3. **Naive-vs-model headcount counter** — naive volume÷baseline-AHT math under-staffs
   (defaults: 31 vs 44, a 13-person gap). Shown as a number under the levers.

## Hard rules (do not reopen — see HANDOFF for the full rejected list)
- ❌ No cost-optimal-coverage marker / U-shaped cost curve. Concavity makes an interior
  minimum **mathematically impossible**. Never draw one.
- ❌ No fuzzy "difficulty slider." Two AHT endpoints (easy + hard) only.
- ❌ No full WFM platform (scheduling, rostering, saved scenarios). One screen, one lever.
- The planning lens **reframes, doesn't compute**. It must never claim to diagnose.

## Caveats to state up front (don't let an interviewer surface them first)
The no-sweet-spot proof holds *within the simplified continuous model* (continuous coverage,
smooth difficulty, linear per-unit costs). Real WFM can reintroduce interior optima via
stepwise staffing, minimum shift coverage, vendor tiers/volume discounts, fixed platform
fees, or SLA penalties. Frame as "no sweet spot in the clean model," not a universal law.

## Reference docs (adjacent, authoritative)
- `wfm-capacity-the full screen + model.md` — full derivation, ASCII screen, hand-reconciled sample numbers
- `wfm-capacity-planner-HANDOFF.md` — the design-conversation handoff, decisions locked & rejected

## Verify the math
The model reconciles to the spec's hand-checked sample (defaults, a=0.595): naive **31**,
model **44**, blended **$4.46**, staffing **~$39.2k/wk**. If you change `derive()` in
`index.html`, re-check against the "Sample numbers" table in the model doc.
