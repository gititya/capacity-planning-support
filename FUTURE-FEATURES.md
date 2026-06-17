# FUTURE-FEATURES.md — resume notes (not yet built)

Three candidate features, prompted by Fernando Duarte's article
*"AI Handled 40 Percent. Headcount Does Not Drop 40 Percent."* The article and this
build **agree on the core mechanism** (AI clears easy contacts first → residual queue
is harder → no linear headcount cut). These three features close gaps where the
article models something the build currently ignores.

**Status: spec only. No code written.** Each spec below is enough to resume in a fresh
LLM session. Implement in `index.html` only (single file, plain JS, no build, no TS).

## Ground rules (carry over from CLAUDE.md — do not break)
- One screen, one swept lever (AI coverage). No full WFM platform.
- Two AHT endpoints only (easy + hard). Feature C strains this — flagged below.
- Planning lens reframes, never claims to diagnose.
- No cost-optimal-coverage marker / no U-shaped cost curve (concavity makes an interior
  cost minimum impossible).
- **Every feature ships OFF by default** so the build still reconciles to the
  hand-checked sample: defaults at a=0.595 → naive **31**, model **44**, blended
  **$4.46**, staffing **~$39.2k/wk** (`skills/ai-capacity-planner/references/the_model.md`).

## Current model recap (index.html `derive()`, ~L235)
```
a            = coverage × success
humanTickets = volume × (1 − a)
baseAHT      = (easy + hard) / 2                     ← naive math
resAHT       = easy + (hard − easy) × (1 + a) / 2    ← truncated mean over surviving slice
humanHours   = humanTickets × resAHT / 60
headcount    = humanHours / prodHrs
naiveHC      = humanTickets × baseAHT / 60 / prodHrs
aiCost       = (mode==='attempt' ? volume×coverage : volume×a) × fee
humanCost    = humanHours × rate
blended      = (aiCost + humanCost) / volume
```

---

## Feature A — AI-supervision load  *(priority 1; LOW rule-conflict)*
**Why:** Article's *supervised work* bucket. AI creates net-new human work that scales
with AI volume: QA-ing AI answers, recontact monitoring, handoff audits, knowledge-base
fixes, routing cleanup. The build currently scores every AI-resolved ticket at **$0
ongoing cost**. Adding this *raises the floor* and makes "the floor costs more than
naive math says" even more true — it reinforces the HERO, doesn't fight it.

**New input:** `supMinPerResolution` — human oversight minutes per AI-resolved contact.
Default `0`. UI: new "AI supervision" group, or under "Human handling". Number input,
min 0, step 0.25.

**`derive()` change:**
```js
const supHours   = resolutions * p.supMin / 60;          // resolutions = volume × a
const humanHours = humanTickets * resAHT / 60 + supHours;
```
Everything downstream (headcount, humanCost, blended, floor) flows unchanged.
**Leave `naiveHC` ignoring supervision** — the naive planner forgets it too, which
correctly *widens* the naive-vs-model gap.

**Reconciliation guard:** `supMin = 0` must reproduce naive 31 / model 44 / $4.46.
`supMin > 0` must (a) raise floor headcount (red line at 100% AI) above 0 even when
success = 100%, and (b) widen the naive-vs-model gap counter.

---

## Feature B — Growth / hiring-avoidance reframe  *(priority 2; LOW rule-conflict)*
**Why:** Article's strongest exec point — AI's real payoff is usually **flat headcount
as volume grows**, not a dramatic cut. "A company that grows 30% without growing
support headcount 30% has improved support economics." The build shows only a static
weekly snapshot, which invites the "where's the reduction slide" trap.

**New inputs:**
- `volumeGrowthPct` — projected volume growth (e.g. +30%). Default `0`.
- A small toggle/mode switch: **Snapshot** (current) vs **Growth**.

**Logic (no `derive()` physics change — just a second call):**
```js
const grown   = { ...p, volume: p.volume * (1 + p.growth) };
const future  = derive(p.coverage, grown);
// reframed delta: future.headcount at this coverage  vs  baseline headcount at 0% AI today
const baseToday = derive(0, p).headcount;
```
Reframe the KPI strip / delta line to: *"Serve +30% volume at this coverage with
**X** humans — vs **Y** you'd need at 0% AI. AI buys growth, not a cut."* Keep the
existing snapshot view as default.

**Reconciliation guard:** `growth = 0` must equal the current snapshot exactly.

---

## Feature C — Assisted-work middle bucket  *(priority 3; MEDIUM rule-conflict)*
**Why:** Article's *shifted work* bucket. AI pre-processes (draft/triage/summary) and a
human still finishes — but **faster** than from scratch. The build is binary today:
an AI ticket is either resolved-and-gone (`a`) or bounced back as a full-price human
ticket at residual AHT. There's no faster-because-AI-helped middle state.

**⚠ Rule tension:** this adds a third AHT regime, which strains the
"two AHT endpoints only" hard rule. **Decide at implementation time** whether to
proceed, or reshape it as a single `assistAHTfactor` multiplier (cheaper, keeps two
endpoints). Do not silently expand the model surface.

**New inputs:**
- `assistShare` — share of human-handled contacts that arrive AI-assisted. Default `0`.
- `assistAHTfactor` — AHT multiplier for assisted contacts (e.g. `0.6`). Default `1`.

**`derive()` change (split the human queue):**
```js
const assisted    = humanTickets * p.assistShare;
const cold        = humanTickets * (1 - p.assistShare);
const humanHours  = (cold * resAHT + assisted * resAHT * p.assistFactor) / 60;
```
Naive math stays unaware of assist (widens the gap, consistent with A).

**Reconciliation guard:** `assistShare = 0` (or `assistFactor = 1`) must reproduce
naive 31 / model 44 / $4.46.

---

## Gaps deliberately NOT planned (collide with hard rules)
- **Escalation / second-escalation / manager span** — counting managers + escalation
  tiers tips into the refused full-WFM platform.
- **Coverage floor by channel/hour/region/language** — already a text caveat; a true
  minimum-staffing floor is a scheduling concern, out of scope for "one lever".
- **Measurement scorecard** (recontact, handoff success, QA defect-type shift) — this
  is a calculator, not a dashboard. Could live in the README, not the screen.

## When implementing any feature
1. Read `skills/ai-capacity-planner/references/the_model.md` first.
2. Add the input OFF by default; confirm the sample still reconciles before touching UI.
3. Re-verify naive 31 / model 44 / $4.46 / ~$39.2k/wk at defaults.
4. Check no NaN/Infinity at coverage 0% and 100%.
