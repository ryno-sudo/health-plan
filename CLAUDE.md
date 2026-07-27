# Ryno's Health Plan — Context for Claude

Read this first. It's the standing profile + decisions so Ryno never has to repeat himself. Update it whenever a decision changes.

## Who / goals
- Age **45–54**, trains **5:30am, 3 days/week baseline** (4th day optional = repeat Day 1).
- Goal: **get more shredded/defined first, a little bigger second.** Priority muscles: **side/rear delts, traps, biceps, triceps, forearms.**
- Reads on his phone — **prefers larger fonts**; apps have A−/A+ controls, keep base ≥18px.

## Hard constraints (never violate)
- **L5/S1 lower-back injury** → no axial spinal loading: no back squats, no barbell deadlifts/rows from floor, no standing barbell press. Use leg press, chest-supported rows, back-supported pressing, Pallof/dead bug core.
- **Knees niggle** → knee-friendly leg work: moderate-ROM leg press, controlled high-rep leg extensions, no deep loaded knee stress.
- **Does NOT do flat bench press** → incline DB (30°) + cable fly for chest.
- **No apnea (his report):** nose surgery done, sleeps with mouth tape. Daytime sleepiness still ticked — home sleep study remains the fallback if sleep fixes don't land in 3–4 wks.

## Health context (from May 2025 bloods — see Meal-Plan.html §1)
- **eGFR 58 unresolved** — possibly creatine artifact. Cystatin C eGFR + urine ACR is the #1 outstanding test. Until then: creatine 5 g (down from 10), and meal portions carry two protein scenarios (~200 g/day vs 110–130 g/day if CKD confirmed).
- **Transferrin sat 48%** — HFE screen outstanding; liver capped at chicken liver 1×/week.
- Gut: **methanogen overgrowth (IMO), low sIgA, high TMAO** → the whole meal plan is gut-focused; no onion/garlic wk 1 of the reset.
- Stack (prescriber-managed — never adjust doses, only flag questions): TRT + Masteron (Sun/Wed), HCG (Mon/Wed/Fri), Reta (Sun/Wed), GHKCu (AM). **HGH and DSIP are PAUSED (not taking, as of Jul 2026)** — no pre-bed injections currently; don't cite the "9:30pm injection block" as a sleep factor while paused. He cycles peptides, so they may return — update this line when they do.
- **Histamine experiment:** strict low-histamine tier started **Mon 20 Jul 2026** (2–4 wks), then relax to LOW tier. Tracking focus/bloating daily to judge it (he's curious about a histamine↔ADHD-traits link — treat as n=1 experiment, not established science).
- **Sleep:** wakes 2–4am "wide awake, mind racing" = hyperarousal pattern; lights-out was 9:30–10:30pm (too late for 5:30am wake). Plan: whole evening earlier (asleep ~9:15pm), 8:30pm brain-dump, caffeine cut-off 10–11am, 3am stimulus-control rescue. Cannot promise outcomes — track hours in the app. Note: the 2–4am waking happens **without** DSIP/HGH in play (paused), so it's not peptide-rebound; hyperarousal + short window remain the working explanation.

## The files (what's canonical)
| File | Role |
|---|---|
| `Workout-Program.html` | **THE training program + all tracking** (training logs, body metrics, sleep, focus/bloating charts). Single source of truth for training. Self-contained HTML app, localStorage, no build step. |
| `Meal-Plan.html` | 4-week gut meal plan, supps/peptides schedule, shopping lists. |
| `index.html` | Landing page linking the two apps. |
| `Low-Histamine-Recipes.md` | Two-tier (LOW/STRICT) swap recipes — TM7 + slow cooker. |
| `Week1-Strict-Histamine-Plan.md` | Strict week 1 day-by-day (started 20 Jul 2026). |
| `roadmap.md` | Future app build plan. Its §6 routine is superseded by `Workout-Program.html`. |

Older/duplicate routines (roadmap §6 A/B/C, Meal-Plan gym references) are design history — **`Workout-Program.html` wins on any conflict.**

## Deployment (important — avoids repeat pain)
- Netlify is (being) connected to this GitHub repo, deploying branch **`claude/workout-health-roadmap-4dkbb8`**, no build command, publish root.
- **Every push to that branch auto-deploys.** Never ask Ryno to download files and drag to Netlify Drop again. After changing any HTML, just commit + push.
- App data lives in localStorage per-site — if the site URL ever changes, remind him to Export backup on the old URL and Import on the new one.

## Working conventions for Claude
- Commit + push after every meaningful change (that's what deploys it).
- HTML apps: validate embedded JS (`new Function`) and functionally test with the pre-installed Playwright Chromium before pushing.
- Keep everything self-contained (no CDNs), mobile-first, large tap targets.
- Health advice: be honest about uncertainty; never promise medical outcomes; prescriber owns all dosing; GP flags stay visible (eGFR test, sleep study fallback).
