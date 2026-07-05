# Health OS — Product & Build Roadmap

A single personal app that runs your whole protocol: gut-focused meal plan, supplement + peptide scheduling (with dosage escalation, cycling, and reordering), a 5:30am strength program, and a recipe engine for the **Thermomix TM7** and the **slow cooker**. Everything below is grounded in the existing `Meal-Plan.html` and the referenced `Training-Program-12-Week.html`.

> **Not medical advice.** This app organises a protocol *you and your prescriber have already agreed to*. It must never invent doses or peptides. Every escalation is a **suggested prompt to confirm with your prescriber**, never an automatic change. See §11.

---

## 1. Vision

One app, opened at 5:15am and before bed, that answers four questions with zero thinking:

1. **What do I eat today?** (meal plan + the exact recipe, TM7 or slow cooker)
2. **What do I take/inject today, and how much?** (supps + peptides, with the busy Wed-night sequence handled)
3. **What do I train today?** (today's session, last session's numbers, what to beat)
4. **Am I improving, and is anything running out?** (progress + inventory + reorder + bloodwork due)

Design principle: **glanceable and decisive.** No dashboards to interpret at 5am. Big "today" card, one tap to log, everything else is secondary.

---

## 2. User context (drives every default)

This is a single-user app for a specific, complex protocol. Hard-code these as the seed profile; make them editable.

| Domain | Facts the app must respect |
|---|---|
| **Training window** | 5:30am, 3–4 days/week. Sessions must fit ~45–60 min. |
| **Injury** | L5/S1 — spinal-loading exercises need safe substitutions flagged. |
| **Gut** | Methanogen overgrowth (IMO), low sIgA, elevated TMAO. Drives food AVOID list + supp priorities. |
| **Kidney flag** | eGFR 58 unconfirmed (possible creatine artifact). **Protein has TWO scenarios** until cystatin C eGFR confirmed — the app must carry both and switch on one setting. Creatine dropped 10g→5g. |
| **Iron flag** | Transferrin sat 48% — liver limited to chicken liver 1×/week until HFE screen clears. |
| **Hormone/peptide stack** | TRT, Masteron, HCG, Reta, HGH, GHKCu, DSIP — each on its own schedule (see §5). |
| **Equipment** | Home + Revo Charlestown gym; TM7; slow cooker. |
| **Goal mode toggle** | "Lean (Reta on)" vs "Growth (Reta paused, +300–400 kcal)" — flips macros app-wide. |

---

## 3. Feature modules (MVP scope in bold)

1. **Today view** — the home screen: meals, doses/injections, session, "running low" chips.
2. **Meal Plan** — 4-week phased schedule, macro scenarios, AVOID list, per-meal recipes.
3. **Recipe engine** — TM7 + slow cooker, scalable, batch-prep aware.
4. **Supps & Peptides scheduler** — the hard part: escalation, cycling, inventory, reorder.
5. **Training program** — the 5:30am split (§6), logging, progressive overload.
6. **Progress & bloodwork** — weekly metrics, trends, milestone lab reminders.
7. Shopping lists — auto-generated from the week's meals + low inventory (Phase 2).
8. Notifications — dose times, Wed-night sequence, "reorder now", bloodwork due (Phase 2).

---

## 4. Data model (the spine)

Keep it local-first. This is health data for one person — **no cloud account required**. SQLite (or IndexedDB via a wrapper) on-device; optional encrypted export.

```
Protocol            one active protocol, holds goal-mode + proteinScenario flag
Item                supp | peptide | medication  (see fields below)
DoseSchedule        per Item: amount, unit, route, timing, day-pattern, with-food
DoseEvent           a logged take/injection (timestamp, amount, site, skipped?)
EscalationRule      per Item: "after N weeks at dose X → suggest dose Y (confirm w/ prescriber)"
Cycle               per Item: onWeeks / offWeeks (e.g. Allicin 2 on / 2 off)
Inventory           per Item: qtyOnHand, unitSize, reorderThreshold, vendor, url, leadTimeDays
Meal                slot (breakfast/lunch/dinner/snack), phase-week, macros, recipeId
Recipe              method (TM7 | slowcooker | stovetop), steps[], scaleFactor, batchYield
TrainingDay         A | B | C, exercises[]
Exercise            name, sets, repRange, load, substitution (L5/S1-safe), muscleTargets[]
SetLog              per exercise per session: weight, reps, rpe, date
Metric             bodyweight, waist, bloating(1-10), sleep(1-10), transit, sickDays
LabMarker           name, lastValue, lastDate, retestIntervalWeeks, priority
```

### `Item` fields (peptides need extra)
```
id, name, category(supp|peptide|med), currentDose, unit, route(oral|subQ|IM|topical),
timing[](AM|lunch|preBed|preWorkout), dayPattern(daily | Mon/Wed/Fri | Sun/Wed | 2on2off),
withFood(bool), site(rotating list for injectables), notes,
active(bool), pausedReason, escalationRuleId?, cycleId?, inventoryId?
```

**Why this shape:** peptides are the reason this app can't be a checklist. A peptide can (a) escalate on a timer, (b) cycle on/off, (c) run out and get **swapped for a different peptide**, and (d) be paused for a medical reason. The `Item`/`DoseSchedule`/`EscalationRule`/`Cycle`/`Inventory` split lets all four happen without rewriting history — `DoseEvent` rows stay immutable so progress charts remain honest.

---

## 5. Supps & Peptides scheduler — the core engine

This is the module that justifies building an app instead of using the HTML file. Four behaviours:

### 5.1 Daily/weekly scheduling
Each `Item` renders into Today view by `timing` + `dayPattern`. Seed data from `Meal-Plan.html` §6:

- **AM (daily):** HGH, GHKCu, Vit D3 2,000 IU, B-complex, Creatine **5g** (reduced), PHGG, Mito Xcell, bone broth + L-glutamine 5g, colostrum (split dose).
- **With lunch (wk 2–4):** Allicin (cycled), Curcumin+piperine 500mg, digestive enzymes + HCl.
- **Pre-bed:** HGH, DSIP, Mg glycinate 400mg, Zinc carnosine 75mg, L-glutamine 5g, Greek yoghurt + collagen.
- **Injection day-patterns:** HCG Mon/Wed/Fri · Reta Sun/Wed · TRT+Mast Sun/Wed.
- **Wednesday-night sequence** (busiest — render as an *ordered, timed checklist*, not a flat list):
  1. Dinner ~6pm (lighter)
  2. ~7:30 HCG sub-Q (site 1)
  3. ~8:00 Reta sub-Q (opposite abdomen)
  4. ~8:00 TRT + Mast IM (glute)
  5. ~9:30 HGH + DSIP sub-Q (separate sites)
  6. Pre-bed yoghurt + Mg + zinc carnosine + glutamine

  → Injection-site rotation tracker so the same site isn't hit twice running.

### 5.2 Dosage escalation ("increase after a certain time")
`EscalationRule` = `{ afterWeeksAtDose, fromDose, toDose, requiresPrescriberConfirm: true }`.

- The engine watches how long an `Item` has been at `currentDose` (from `DoseEvent` history).
- When the threshold hits, it raises a **suggestion card**: *"You've been on Reta 0.25mg for 4 weeks. Plan allows titration to 0.5mg. Confirm with prescriber before increasing."* → buttons: **Confirm & update dose** / **Not yet** / **Snooze 1 week**.
- Confirming writes a new `currentDose` and starts the clock again. History is preserved.
- Ships with the built-in escalations already in the plan (e.g. the wk-2 "escalation if gut not improving" additions: Atrantil, MegaSporeBiotic wk4+, Berberine if IMO confirmed) as **conditional** rules gated on a progress signal, not a pure timer.

### 5.3 Cycling on/off
`Cycle = { onWeeks, offWeeks, startDate }`. Allicin ships as **2 on / 2 off**. During "off" weeks the Item greys out of Today view and shows a countdown to next "on". Generalised so any peptide can be cycled (common for GHK-Cu, HGH-secretagogue blends, etc.).

### 5.4 Inventory, cycling-out & reorder ("change peps as they run out / buy new")
This is the feature you explicitly asked for.

- Every `Item` has `Inventory { qtyOnHand, unitSize, reorderThreshold, vendor, url, leadTimeDays }`.
- Each logged `DoseEvent` **decrements** `qtyOnHand` by the dose.
- Runway = `qtyOnHand / avgDailyUse`. When runway ≤ `leadTimeDays + buffer`, Today view shows a **"Running low"** chip and a **Reorder** button (opens vendor URL — iHerb links already in the shopping data; add your peptide vendor).
- **Swap flow** (peptide runs out and you replace it with a different one): "Retire / Replace" on an Item →
  - marks the old Item `active:false` (history kept),
  - clones its `DoseSchedule`/`timing`/`site-rotation` into the new Item so you don't re-enter it,
  - prompts for new dose + vendor + qty.
- **Cycle-off flow:** "Pause & cycle off" sets `active:false` with `pausedReason`, keeps it in a **"Currently off"** shelf with a *resume* button, and stops decrementing inventory.

**Acceptance test for this module:** run a simulated 6 weeks — a peptide escalates once, another cycles 2-on/2-off, a third depletes and triggers reorder, a fourth is swapped for a replacement — and Today view + progress charts stay correct with no manual data fix-ups.

---

## 6. The killer 5:30am routine

Built for **3–4 mornings/week, ~50 min, L5/S1-safe, all muscle heads hit**. A 3-day rotation (A/B/C); on 4-day weeks you run A/B/C then repeat A. Each session: 5-min warm-up → main work → 2-min finisher. Rep tempo controlled; every exercise has an L5/S1-safe note where spinal loading is a risk.

**Warm-up (all days, 5 min):** bike/row 3 min + banded shoulder dislocates + hip airplanes + glute bridges (protect L5/S1 — brace before every working set).

### Day A — Push / Quad focus (7 exercises)
| # | Exercise | Sets × Reps | Notes / L5/S1 |
|---|---|---|---|
| A1 | Machine Chest Press or Flat DB Press | 3 × 6–10 | Supported back; skip barbell bench if it aggravates arch |
| A2 | Leg Press (feet mid-high) | 3 × 8–12 | **Replaces back squat** — no axial load on L5/S1 |
| A3 | Incline DB Press | 3 × 8–12 | Upper chest |
| A4 | Seated DB Shoulder Press | 3 × 8–12 | Back supported |
| A5 | Cable Triceps Pushdown | 3 × 10–15 | — |
| A6 | Wrist Curl | 3 × 12–15 | **Moved here from Day C** — fresh forearms; presses don't pre-exhaust them |
| A7 | Pallof Press (anti-rotation core) | 3 × 10/side | L5/S1-safe core; no spinal flexion |

### Day B — Hinge / Legs / Posterior chain (7 exercises)
| # | Exercise | Sets × Reps | Notes / L5/S1 |
|---|---|---|---|
| B1 | Romanian Deadlift (light–moderate) or Hip Thrust | 3 × 8–12 | **Hip thrust preferred** if RDL loads the spine; hamstrings + glutes |
| B2 | Walking Lunge or Bulgarian Split Squat | 3 × 8–10/leg | Unilateral, low spinal load |
| B3 | Seated Leg Curl | 3 × 10–12 | Hamstring isolation |
| B4 | Leg Extension | 3 × 12–15 | Quad isolation |
| B5 | Standing Calf Raise | 4 × 12–15 | — |
| B6 | Cable Crunch / Dead Bug | 3 × 12 | Controlled; dead bug if crunch bothers back |
| B7 | Hanging Knee Raise | 3 × 10–12 | Decompresses spine |

### Day C — Pull / Back / Traps (10 exercises)
| # | Exercise | Sets × Reps | Notes |
|---|---|---|---|
| C1a | Straight-Arm Pulldown | 3 × 12–15 | Lower-trap + lat pre-activation |
| C1b | Lat Pulldown | 3 × 8–12 | Superset with C1a; upper-trap stabiliser |
| C2 | Chest-Supported Row | 3 × 8–12 | **Chest-supported = zero low-back load**; mid/lower traps |
| C3a | Face Pull | 3 × 15 | Rear delt + mid trap |
| C3b | Rear-Delt Fly | 3 × 12–15 | Superset with C3a |
| C4 | DB Shrugs (3-sec hold) | 3 × 10–12 | Direct upper-trap isolation |
| C5 | DB Biceps Curl | 3 × 8–12 | — |
| C6 | Hammer Curl | 3 × 10–12 | *(was C7)* brachialis/forearm |
| C7 | Cable Rope Curl or Incline Curl | 3 × 12–15 | *(was C8)* |
| — | ~~Wrist Curl~~ | — | **Removed from Day C → now A6** |

> **Traps are covered from five angles** in Day C already: C4 shrugs (upper), C1b lat pulldown (upper, as stabiliser), C2 row (mid/lower), C3a face pull (mid), C1a straight-arm pulldown (lower). No extra trap work needed until 4–6 weeks in; only then consider adding shrug volume or a cable upright row — and only if upper traps specifically lag.

**Progressive overload rule the app enforces:** when you hit the top of the rep range for all sets, it prompts a load increase next session (double-progression). It surfaces last session's weight×reps per exercise so you always know the number to beat. Deload every 6th week (−40% volume) — auto-scheduled.

---

## 7. Meal plan & recipe engine

### 7.1 Meal plan (from `Meal-Plan.html`)
- **4-week phased schedule:** Wk1 strict gut reset → Wk2 reintroduce variety (onion/garlic back in *cooked*) → Wk3–4 full variety.
- **Two macro scenarios** (kidney-gated): normal-protein ~200g/day vs CKD-protein 110–130g/day. One toggle flips every meal's portions and the shopping list.
- **Goal-mode toggle:** Lean (Reta on) vs Growth (Reta paused, +300–400 kcal → ~2,800 kcal).
- **AVOID list surfaced contextually** (methanogen/gas triggers) — show it on any recipe that would otherwise tempt a swap.
- **Reta-day eating pattern** shown on Sun/Wed automatically.

### 7.2 Recipe engine — TM7 + slow cooker
Each `Recipe` carries a `method` and renders method-specific steps. Seed with the existing slow-cooker recipes (beef & root-veg stew, slow lamb shoulder) and **add TM7 versions**.

- **TM7 recipes** store structured steps: `{ time, temp, speed, action }` so each step reads like a Thermomix guided step (e.g. *"Sauté onion — 3 min / 120°C / speed 1"*). Include TM7-native builds of plan meals: bone broth, pumpkin/carrot soups, sauces, protein-rich risottos, hummus-free dips (oxalate-aware), sorbets for the yoghurt swap.
- **Slow-cooker recipes** store `{ setting(low|high), hours }` + the "set it Sunday morning" batch note.
- **Both methods scale** by portion count and are **batch-prep aware** (the plan assumes ~90 min Sunday prep + one 45-min weeknight). A "Sunday Prep" screen aggregates every batch task for the week (slow-cook lamb Sat night, cook 4–6 cups rice, roast veg tray, 6 hard-boiled eggs) into one checklist.
- **Tolerance flags** carried onto recipes: bone broth (histamine/glutamate), pomegranate juice (fructose), yoghurt (bloating), oxalate caution.

**Recipe acceptance test:** pick any plan meal → app offers a TM7 route *and* a slow-cooker/stovetop route where sensible, scaled to the chosen portions, with the correct tolerance flags shown.

---

## 8. Progress & bloodwork

- **Weekly log (Sat fasted):** bodyweight, waist@navel, bowel transit, bloating 1–10, sleep 1–10, sick-days count (sIgA proxy), top-set lifts.
- **Charts:** each metric over time; overlay protocol changes (dose escalations, cycle on/off, Reta pause) as vertical markers so cause/effect is visible.
- **Bloodwork milestones** as scheduled reminders in priority order: (1) cystatin C eGFR + urine ACR — gates the protein scenario; (2) HFE screen + iron studies — gates liver frequency; (3) ApoB + Lp(a); (4) FBC/Hct q3mo on TRT; (5) HbA1c; (6) PSA quarterly; (7) repeat gut-mapping at 12 wk.
- Confirming a lab result can **auto-flip a plan setting** (e.g. cystatin C confirms kidney OK → switch protein scenario back to 200g and restore creatine 10g, *with a confirm step*).

---

## 9. Tech stack (recommendation)

Optimise for: single user, health-private, offline at 5am, fast to build, cross-device (phone-first).

- **App:** React + Vite PWA (installable, works offline, one codebase phone+desktop). The existing artifact is already HTML/JS — a PWA is the shortest bridge.
- **State/data:** local-first — **SQLite via `sql.js`/`wa-sqlite`** or **Dexie (IndexedDB)**. No backend needed for MVP.
- **Persistence today:** the current file uses `localStorage` for checkboxes — fine for MVP, migrate to Dexie when the scheduler lands.
- **Notifications:** PWA Notifications API + Periodic Background Sync for dose/reorder/bloodwork nudges (fallback: local scheduled notifications).
- **Optional sync/backup:** encrypted JSON export/import first; only add a backend (e.g. Supabase) if multi-device sync is wanted — keep it opt-in given the data sensitivity.
- **Charts:** lightweight (uPlot or Recharts).

Keep everything client-side and private by default. No third-party analytics on health data.

---

## 10. Build phases

**Phase 0 — Foundation (1 wk)**
Scaffold PWA, data model (§4), migrate current HTML content in, Dexie/SQLite layer, Today view shell.

**Phase 1 — MVP (2–3 wk)** — the four "5am questions"
- Today view (meals + doses + session + low-stock chips)
- Supps/peptides daily+weekly scheduling incl. Wed-night ordered sequence + site rotation
- Meal plan with 4-week phases + macro/goal toggles
- Training module: A/B/C days (§6) + set logging + last-session numbers + double-progression prompt
- Weekly progress log

**Phase 2 — The engine (2–3 wk)** — the reason it's an app
- Escalation rules (timer + conditional, prescriber-confirm gated)
- Cycling on/off
- Inventory + reorder + **swap/replace peptide** + cycle-off shelf
- Recipe engine with TM7 structured steps + slow-cooker + scaling + Sunday Prep aggregator

**Phase 3 — Polish (1–2 wk)**
- Auto shopping lists (week meals + low inventory)
- Notifications (doses, Wed sequence, reorder, bloodwork due)
- Bloodwork tracker with auto plan-flip (confirm-gated)
- Charts with protocol-change overlays
- Encrypted export/backup

**Phase 4 — Optional**
Multi-device sync (opt-in), wearable import (sleep/HR), prescriber-shareable PDF summary.

---

## 11. Safety, privacy & guardrails (non-negotiable)

- **Never auto-change a dose or add a peptide.** Every escalation/addition is a *suggestion* requiring an explicit "Confirm with prescriber" tap. Log who/when confirmed.
- **Hard medical flags stay visible:** Masteron + eGFR interaction; cod liver oil + D3 overdose risk; retinol toxicity ceiling (>25,000 IU/day across supp+liver+CLO); B6 doubling (B-complex + Mito Xcell); iron loading (transferrin sat 48%). The app should *warn on conflicts*, e.g. if vit A supp + cod liver oil + weekly liver are all active at once.
- **Injectables:** site-rotation enforced; no medical dosing math beyond what's entered; clear "this is your prescriber's protocol" framing.
- **Privacy:** local-first, no account required, no analytics on health data, encrypted export only.
- **Disclaimer on first run and on the supps/peptides screen:** organiser only, not medical advice; confirm stack + escalations + eGFR finding with GP/integrative practitioner.

---

## 12. Open questions to resolve before Phase 1

1. Confirm cystatin C eGFR status — sets the **default** protein scenario (don't guess).
2. Peptide vendor URLs + unit sizes for inventory/reorder seeding (iHerb links already exist for supps).
3. Exact current peptide doses to seed `currentDose` (kept out of this file deliberately — enter in-app).
4. Which plan meals you most want TM7 versions of first (soups/broth/sauces are the highest-value TM7 wins).
5. 3-day vs 4-day training default week, and preferred deload cadence.
```
