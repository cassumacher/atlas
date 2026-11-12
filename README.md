# Atlas

**A single-file workout logger with a built-in coach.** Log every set at the gym, and Atlas tells you what to lift next time, based on what you actually did last time.

Built for a lean-bulk hypertrophy block: 4 training days, progressive overload, and a coach's review after every session.

---

## Features

| Feature | What it does |
|---|---|
| **Today's targets** | Opening a day shows every exercise with its target weight and reps (▲ going up, ▼ easing off, or building reps). Weights are pre-filled into the set rows. |
| **Double-progression coach** | Top of the rep range on every set → add weight. Well above range → jumps to a weight that lands you in range. Below range → drop a step. In range → same weight, add a rep. |
| **Coach's review** | After **Finish session**: new bests (est. 1RM), what goes up next time, too heavy/light, warm-up sets logged as work sets, skipped exercises, volume vs. last session, duration. |
| **Exercise swaps** | Every exercise has 2–3 alternatives in a dropdown (machine taken, equipment missing). Each alternative keeps the same sets/reps/rest and tracks its own progress. |
| **Weight caps** | Exercises with a fixed maximum (e.g. a 30 kg fixed EZ bar) switch to rep and tempo progression instead of suggesting weights that don't exist. |
| **Crash-proof drafts** | Autosaves 2 s after each change and when the app is hidden or closed. **Save progress** forces a save; **Finish session** commits it to History. Reopening restores the exact state, on any device. |
| **Rest timer** | Tap ✓ on a set to start that exercise's rest timer, with +15 s / Skip and vibration when done. |
| **Progress** | Est. 1RM trend per exercise, bodyweight log with 7-day average and weekly rate of gain vs. a 0.25–0.4 kg/week target, with calorie guidance. |
| **Form tips** | Three key cues per exercise plus a link to a demo search. |
| **Light/dark mode, mobile-first** | Designed to be used one-handed between sets. |

## How it works

Atlas is **one HTML file**: React 18 (UMD from cdnjs), no build step, no bundler.

```
index.html        # the whole app: UI, coach engine, charts, timer
plan/plan.json    # the training plan (days, exercises, rep ranges, rest, alternatives)
```

### Data model

Atlas runs as a [Claude](https://claude.ai) artifact and stores data through the artifact runtime's document database (`window.claude.use("db")`):

| Path | Contents |
|---|---|
| `config/plan` | The training plan (`plan/plan.json` is a copy of it) |
| `config/body` | Bodyweight entries `{ date, kg }` |
| `sessions/{date}_{dayId}` | Finished sessions: `{ date, dayId, entries: { exerciseId: [{ w, r }] }, minutes }` |
| `drafts/{dayId}` | In-progress session, deleted on Finish |

Browser `localStorage` is used only as a fast local copy of the active draft; the cloud draft is the source of truth.

> **Running outside Claude:** opened as a plain file, the page renders but shows a notice that storage isn't available. To self-host, replace the `window.claude.use("db")` calls with your own backend (Firebase/Firestore maps almost 1:1: `doc().get/set/delete/onSnapshot`, `collection().orderBy().limit()`), and load `plan/plan.json` into `config/plan`.

### Plan format

```json
{
  "id": "db-bench",
  "name": "Dumbbell bench press",
  "sets": 4, "repMin": 6, "repMax": 10,
  "rest": 120,
  "inc": 2,
  "maxW": null,
  "cues": ["Shoulder blades back and down…"],
  "alts": [{ "id": "machine-chest-press", "name": "Machine chest press", "inc": 2.5 }]
}
```

- `inc`: weight step when progressing (kg)
- `maxW`: optional ceiling; above it the coach switches to rep/tempo progression
- `alts`: inherit sets/reps/rest from the parent exercise

## Logging conventions

- **Barbell:** total weight, bar plus plates on both sides.
- **Dumbbells:** weight of one dumbbell.
- **Machines:** the stack/pin number, or plates added; just be consistent.
- **Bodyweight:** reps only, leave kg empty. Holds (side plank): seconds as reps.

## The plan (block 1)

| Day | Focus |
|---|---|
| A | Legs & shoulders: belt squat, RDL, Bulgarian split squat, leg curl, shoulder press, lateral raise, calves |
| B | Chest & triceps: DB bench, incline DB, butterfly, lateral raise, pushdown, overhead extension, dead bug |
| C | Back & biceps: single-arm pulldown & row, lat pullover, reverse fly, EZ curl, cable curl, toes to bar |
| D | Lower body 2 (optional 4th day): leg press, hip thrust, seated leg curl, calves, Pallof press, crunch |

*This plan is not medical advice. Adjust exercises to your own body and any guidance from your doctor or physio.*
