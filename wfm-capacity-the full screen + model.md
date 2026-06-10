# AI-First Capacity Planner — Model & Screen (rev. 2026-06-10)

> "More automation isn't always fewer people — and it has no cost sweet spot."

A single-screen capacity planning prototype. Sweep one lever (AI coverage),
watch two things move (headcount + blended cost per contact), and read a
planning lens that interprets *why* the leftover work is slow.

It's the **union** of three things:
- **A published industry staffing model** — the staffing mechanics (residual volume → headcount). Stops at "how many."
- **A blended-cost lens** — blended AI-vs-human cost per contact. Asks "is it worth it."
- **The builder's mechanism** — residual AHT rises as AI takes more, because the
  easy tickets leave the pool first. This is what makes the headcount floor more
  expensive than naive math predicts.

Built as a **transparent simulator**, not a demo with preordained curves. Every
number traces to a visible formula. The interviewer can change inputs and see
when the thesis holds or breaks.

---

## What changed from the first sketch (and why)

The original "locked" model overclaimed the math. Three corrections, all making
the artifact more honest and more defensible:

1. **Residual AHT now has an explicit formula and an explicit difficulty-shape
   assumption.** Two AHT endpoints alone don't determine the residual average —
   you also need to state how ticket difficulty is distributed. We state it: a
   straight-line ramp, easy tickets first.
2. **The blended-cost "sweet spot" was fiction and is removed.** Under any
   increasing difficulty curve, blended cost-per-contact is *concave* in
   coverage — it can fall, rise, or rise-then-fall, but it can **never** dip to a
   low point and curve back up. An interior cost *minimum* is mathematically
   impossible. So we never draw one. (Proof below.)
3. **Headcount-floor leads as the hero**, because it's the claim that survives
   any slider setting. Blended cost is the honest second chart, not the thing
   the artifact depends on.

---

## The difficulty model (explicit, simple)

Line every ticket up from easiest to hardest on a 0→1 scale. Handling time rises
in a straight line from the easy endpoint to the hard endpoint:

```
aht(p) = Easy + (Hard − Easy) · p          for p in [0, 1]
```

AI resolves the **easiest** fraction first. Effective automation
`a = coverage × success` is the share of all tickets AI actually resolves, so AI
clears `[0, a]` and humans keep the harder slice `[a, 1]`.

**Stated simplification (don't hide it):** we model the *net* effect as "AI
resolves the easiest `a` fraction." In reality some AI attempts fail and escalate,
and those escalations aren't strictly the hard ones — but the easy-first
abstraction is the honest first-order story and keeps the formula reproducible.

> **FYI — known limitation, do NOT build around it.** The easy-first rule treats
> the human pile as only "the tickets AI never touched." But AI also *attempts*
> some hard tickets and *fails* — and failures are plausibly skewed *harder* than
> average (that's likely why they failed). So the real residual pile is probably a
> bit harder than this model assumes, meaning we're if anything slightly
> *understating* the human cost/headcount — the model errs conservative, not
> flattering. Capturing the skew would mean tracking attempts vs. resolutions
> separately and modeling difficulty among failures — too much machinery for a
> one-screen artifact, and it doesn't change the headline. We keep the clean
> version on purpose. **Don't implement this; just be ready to name it first if an
> interviewer probes the edges.**

---

## The model (every line reproducible)

```
a (effective automation) = coverage × success
human tickets            = volume × (1 − a)

baseline AHT             = (Easy + Hard) / 2                 ← all-ticket mean; naive math uses this
residual AHT             = Easy + (Hard − Easy) · (1 + a) / 2   ← average over the hard slice [a,1]

human hours              = human tickets × residual AHT / 60
headcount                = human hours / productive hrs per agent

attempts                 = volume × coverage
resolutions              = volume × a
AI cost (one mode only):
   per-attempt mode      = attempts    × AI $/attempt        (resolution fee = 0)
   per-resolution mode   = resolutions × AI $/resolution     (attempt fee   = 0)

human cost               = human hours × human $/hr
blended $/contact        = (AI cost + human cost) / volume

fully-loaded $/agent/wk  = human $/hr × productive hrs per agent   ← shown next to headcount
```

### Where the residual-AHT formula comes from

Average handling time over the surviving slice `[a, 1]`:

```
residual AHT = (1 / (1 − a)) · ∫ₐ¹ [Easy + (Hard − Easy)·p] dp
             = Easy + (Hard − Easy) · (1 + a) / 2
```

Sanity check: at `a = 0` it equals the all-ticket mean `(Easy + Hard)/2`; as
`a → 1` it equals `Hard`. The survivors get slower on their own as AI eats the
easy end. No tuned slider, no magic ramp.

---

## Inputs (levers)

| Input | Default | Note |
|---|---|---|
| Weekly volume | 10,000 | |
| AI coverage % | 70% | the swept lever |
| AI success % | 85% | sets the headcount floor |
| Easy-ticket AHT | 4 min | easy endpoint |
| Hard-ticket AHT | 25 min | hard endpoint |
| Productive hrs/agent/wk | 32 | |
| Human $/hr | $28 | per-hour cost; per-agent/wk = $896 shown alongside |
| **AI billing mode** | per resolution | radio: **per attempt** *or* **per resolution** — the other fee is zero |
| AI fee | $0.90 | interpreted per the selected mode |

The billing toggle matters: **leading AI agents charge per resolution**, so set the
mode to per-resolution to mirror a real product; per-attempt mirrors vendors that
bill every conversation AI touches.

---

## HERO — Output 1: Headcount (and why it can't go to zero)

Two separate facts, kept separate:

1. **The floor exists because AI success < 100%.** Even at 100% coverage,
   `a = success`, so `volume × (1 − success)` tickets always bounce to a human.
   That floor is set by the success rate, not by coverage — pushing coverage
   higher can't remove it.
2. **The floor is more expensive than naive math says**, because the survivors
   are the hard tickets. Naive math multiplies the floor by *baseline* AHT;
   honest math multiplies by *residual* AHT, which is larger.

This is the most defensible claim in the artifact: it depends only on imperfect
AI + harder survivors. No price assumptions, no curve shape that might bend the
wrong way. Robust across normal imperfect-AI / mixed-difficulty settings — it
degenerates only at the edges (success = 100% removes the floor; easy AHT = hard
AHT removes the naive gap).

---

## Output 2: Blended cost / contact (emergent, no sweet spot)

**Why there is no interior minimum.** Per contact:

```
blended(a)  = (AI marginal term, linear in a) + (W/60) · ∫ₐ¹ aht(p) dp
d/da        = (AI marginal cost) − (W/60) · aht(a)
d²/da²      = −(W/60) · aht′(a)
```

Difficulty is increasing, so `aht′ ≥ 0`, so the second derivative is `≤ 0`
everywhere: blended cost is **concave in coverage**. A concave curve's only
interior extremum is a *maximum*. Therefore an interior cost *minimum — a "best
automation level" — cannot exist*, for any prices and any increasing difficulty
curve. Adding a second AI fee (attempt + resolution) only adds linear terms and
doesn't change this.

**Scope of the proof (state it up front).** This holds *within the simplified
continuous model*: continuous coverage, smooth difficulty, linear per-unit costs.
Real WFM can reintroduce interior optima via stepwise staffing, minimum shift
coverage, vendor tiers / volume discounts, fixed platform fees, or SLA penalties.
Frame the claim as "no sweet spot in the clean model," not as a universal law —
and name those caveats before an interviewer does.

Intuition: AI eats the cheap tickets first, so each extra slice of automation
displaces a *more* valuable human ticket than the slice before — savings per step
grow, they don't shrink. A valley would require shrinking savings. Impossible.

**The three real regimes** (the marker shows whichever the inputs land in):

| Condition | Curve | Marker |
|---|---|---|
| AI marginal cost ≤ human cost of the *easiest* ticket | falls throughout | **Cheapest at MAX coverage** |
| AI marginal cost ≥ human cost of the *hardest* ticket | rises throughout | **Cheapest at ZERO coverage (AI never pays)** |
| in between | rises then falls | **WORST at X% — cheapest at an end** |

Never an interior minimum. The falsifiable claim handed to the interviewer:
*"there is no automation sweet spot on cost — it's monotone, or it's
worst-in-the-middle, never best-in-the-middle. Break it with the sliders."*

---

## The screen

```
┌─────────────────────────────────────────────────────────────────────┐
│  AI-FIRST CAPACITY PLANNER                                            │
│  "More automation isn't always fewer people — and has no cost optimum"│
├──────────────────────────┬────────────────────────────────────────── │
│  LEVERS                   │   OUTPUT 1 — HEADCOUNT  (HERO)            │
│                           │                                           │
│  Weekly volume   10,000   │   people                                  │
│  AI coverage    ▓▓▓▓░ 70% │   75│●..                                  │
│  AI success     ▓▓▓▓▓ 85% │     │   ''●..                             │
│  Easy AHT        4 min     │   44│        ''●●..                       │
│  Hard AHT       25 min     │     │             ''●●  ← floor ≈18 set  │
│  Hrs/agent/wk    32        │   18│··················  by (1−success), │
│  Human $/hr      $28        │    0└─────────────────  costs more than │
│   (= $896/agent/wk)        │      0%  coverage  100%   naive thinks   │
│  AI billing:               │                                           │
│   ( ) per attempt          ├─────────────────────────────────────────  │
│   (•) per resolution       │   OUTPUT 2 — BLENDED $ / CONTACT          │
│  AI fee         $0.90      │                                           │
│                           │   $  │●.                                   │
│  ───────────────────────  │      │  '●..                              │
│  Automation rate:  59.5%  │      │     ''●●..                         │
│                           │      │          ''●●●●  ← falls throughout │
│  Naive math says:  31 ppl │      └──────────────────  CHEAPEST AT MAX │
│  This model says:  44 ppl │        0%  coverage  100%  (this default) │
│  ▲ you'd under-staff 13   │                                           │
│  Model staffing: $39.4k/wk│                                           │
├──────────────────────────┴────────────────────────────────────────── │
│  PLANNING LENS — IF AHT IS HIGH HERE, WHICH IS IT?  (not computed)    │
│   ( ) Structural — work is genuinely harder → accept, staff for it     │
│   ( ) Capability — agents struggling        → train, don't add heads   │
│   ( ) Load — context-switching inflating it → fix concurrency first    │
│   (interpretation lens, not a diagnosis — no input proves which one)  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Sample numbers (reconcile-by-hand, defaults above, a = 0.595)

| Quantity | Value | From |
|---|---|---|
| baseline AHT | 14.5 min | (4 + 25)/2 |
| residual AHT | 20.75 min | 4 + 21·(1.595)/2 |
| human tickets | 4,050 | 10,000·(1 − 0.595) |
| human hours | 1,400.5 | 4,050·20.75/60 |
| **naive headcount** | **31** | 4,050·14.5/60 ÷ 32 |
| **model headcount** | **44** | 1,400.5 ÷ 32 |
| under-staff gap | **13** | 44 − 31 |
| attempts | 7,000 | 10,000·0.70 |
| resolutions | 5,950 | 10,000·0.595 |
| AI cost (per-resolution, $0.90) | $5,355 | 5,950·0.90 |
| AI cost (per-attempt, $0.90) | $6,300 | 7,000·0.90 |
| human cost | $39,214 | 1,400.5·28 |
| blended $/contact (per-resolution) | $4.46 | (39,214 + 5,355)/10,000 |
| blended $/contact (per-attempt) | $4.55 | (39,214 + 6,300)/10,000 |
| fully-loaded $/agent/wk | $896 | 28·32 |
| model staffing $/wk | $39.4k | 44·896 |

The old sketch's "20 vs 34" and "cost-optimal 64%" were placeholders — discarded.

---

## The planning lens (was "diagnosis panel")

It does **not** compute and must not claim to. The same residual-AHT number can
mean three different things, each with a different action:
- **Structural** — work is genuinely harder → accept it, staff for it.
- **Capability** — agents are struggling → train, don't add heads.
- **Load** — context-switching inflates handling time → fix concurrency first.

No input in this model proves which one is true, so it's framed as an
interpretation/planning lens, not a diagnosis. (If we later add real evidence
signals — e.g. AHT variance, tenure mix, concurrency telemetry — it could become
a computed diagnosis. Not in scope now.)

---

## The moments it creates

1. **Headcount drops to a floor, not to zero** — and the floor costs more than
   naive math expects. The hero. Robust to every slider.
2. **No cost sweet spot** — blended cost is monotone or worst-in-the-middle,
   provably never best-in-the-middle. The falsifiable dare.
3. **Naive-vs-model counter** — the old volume÷average-AHT math under-staffs;
   show the gap (here, 13 people) as a number.

---

## Build defaults when greenlit

Plain HTML + JS, single file, no framework, no TypeScript, no build step. Two
line charts. Left: number inputs + the billing-mode radio. Right: headcount chart
(hero, on top) then blended-cost chart with the live regime marker. Planning lens
across the bottom. The naive-vs-model counter and fully-loaded $/agent/wk read out
beneath the levers.
