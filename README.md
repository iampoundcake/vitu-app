# vitu-app

# Fitness App – MVP Product Requirements Document (v0.1)

---

## 1. Problem

Re‑calculating daily calories, macros, and training volume is complex and time‑consuming for most people. Hiring a coach is expensive, and existing apps are static or overwhelming. Users—especially those in life transitions—need pro‑level adaptive guidance **without spreadsheets, over‑complication, or $$$**.

---

## 2. Personas

| Persona          | Trigger Context                 | Primary Job‑To‑Be‑Done                 |
| ---------------- | ------------------------------- | -------------------------------------- |
| Beginner         | Starting fitness journey        | “Give me a fool‑proof plan.”           |
| Lifestyle Lifter | Wants sustainable progress      | “Optimize without spreadsheets.”       |
| Event Preparer   | Peaking for a date‑driven goal  | “Dial in everything, now.”             |
| Transitional     | New baby / postpartum / new job | “Keep me on track when life is chaos.” |

---

## 3. Value Proposition

**“Pro‑level adaptive coaching in your pocket—no math, no spreadsheets, no coach fees.”**

---

## 4. MVP Must‑Have Features

1. Dynamic macro engine (daily & weekly)
2. Adaptive workout generator
3. Goal modes (Beginner / Normal / Hard)
4. Wearable import – Apple Health, Google Fit; Garmin Connect **moved to Phase 1**
5. Shareable daily progress card (PNG)
6. Customizable end‑of‑day **Habit Tracker** (feeds coaching tips)

*Out‑of‑Scope V1*: nutrition database, exercise video library, social feed/leaderboard, desktop companion.

---

## 5. Habit Tracker (MVP Cut)

- **Preset Habits:** Creatine, Water goal, Electrolytes, Meditation, + Custom
- **UX:** end‑of‑day modal checklist → missed habits show a subtle badge on Home.
- **Data:** `habit_id`, `date`, `status` (boolean)
- **Coach Logic:** if habit skipped ≥ 3 consecutive days ⇒ inject tip on next Coach card.
- **Share Card:** optional “✅ X/Y habits” footer.

### Default Habit Library (15 presets, cap 5 active)

| ID | Habit              | Default Target              | Type       |
| -- | ------------------ | --------------------------- | ---------- |
| 1  | Creatine           | 5 g daily                   | binary     |
| 2  | Water Intake       | ½ × bw(lbs) → oz            | numeric    |
| 3  | Electrolytes       | ≥ 1 serving                 | binary     |
| 4  | Meditation         | ≥ 5 min                     | time       |
| 5  | Breath‑work        | 3 min                       | time       |
| 6  | Morning Sunlight   | ≥ 10 min before 10 am       | time       |
| 7  | Mobility / Stretch | ≥ 10 min                    | time       |
| 8  | Daily Walk         | ≥ 20 min **or** 7 k steps   | time/steps |
| 9  | Protein Goal Met   | ≥ macro goal                | numeric    |
| 10 | Veggie Quota       | Veg ≥ 2 meals               | binary     |
| 11 | Caffeine Cut‑off   | No caffeine after 14:00     | binary     |
| 12 | Screen Curfew      | No screens ≥ 30 min pre‑bed | binary     |
| 13 | Gratitude Journal  | 3 items                     | binary     |
| 14 | Cold Shower        | 1–3 min                     | time       |
| 15 | Consistent Bedtime | In bed ± 30 min target      | binary     |

#### Gamification Logic – “Habit Survival Mode”

1. **Start Size:** Pick 1–5 habits during onboarding; extra slots locked.
2. **Strike System:** Lose habit if missed 2 days straight **or** > 2 times in 30 days.
3. **Lost Habit Behaviour:** Habit disappears; Coach card nudges rebuilding.
4. **Unlock Rule:** Need 7‑day perfect streak on remaining habits to add/restore.
5. **Share Card:** Shows only active habits (`✅ n/n`).

---

## 6. Algorithms

### 6.1 TDEE Calculation

1. **Convert units**
   - `weight_kg = weight_lbs / 2.2046`
   - `height_cm = ((feet * 12) + inches) * 2.54`
2. **Mifflin‑St Jeor**
   - **Men:** `(10 × weight_kg) + (6.25 × height_cm) − (5 × age) + 5`
   - **Women:** `(10 × weight_kg) + (6.25 × height_cm) − (5 × age) − 161`
3. **Activity Multiplier:**
   - Sedentary: 1.2
   - Light: 1.375
   - Moderate: 1.55
   - Very Active: 1.725
   - Extra Active: 1.9

### 6.2 Caloric Adjustment

| Goal        | Formula            |
| ----------- | ------------------ |
| Cutting     | `TDEE × 0.75–0.85` |
| Maintenance | `TDEE`             |
| Bulking     | `TDEE × 1.10–1.20` |

### 6.3 Macro Breakdown

| Goal        | Protein % | Carbs % | Fat % |
| ----------- | --------- | ------- | ----- |
| Cutting     | 30–40     | 30–40   | 20–30 |
| Maintenance | 25–35     | 35–50   | 20–30 |
| Bulking     | 25–30     | 40–55   | 20–30 |

**Grams Conversion**

```
protein_g = (calories × protein_pct) / 4
carb_g    = (calories × carb_pct) / 4
fat_g     = (calories × fat_pct) / 9
```

### 6.4 Volume & Deload Rules

- **Base weekly sets:** 10–12 per muscle
- **Progression:** +1 set/week up to 3 weeks, cap ~18
- **Deload:** Week 4 at −40 %
- **Readiness Modifiers:**
  - Sleep < 6 h: ×0.85
  - HRV z‑score ≤ −1 or Resting HR +10 bpm: ×0.90
  - DOMS ≥ 4/10: ×0.90
  - Stress ≥ 7/10: ×0.95
- **Clamp** between 0.65 and 1.10

### 6.5 Nutrition & Fatigue Coupling

| Phase  | kcal Target | Protein      | Carbs    | Fat        |
| ------ | ----------- | ------------ | -------- | ---------- |
| Cut    | −20% eTDEE  | 1.8–2.2 g/kg | 3–4 g/kg | 0.8–1 g/kg |
| Recomp | eTDEE       | 1.6–2.0 g/kg | 4–5 g/kg | 1 g/kg     |
| Bulk   | +10% eTDEE  | 1.6–2.0 g/kg | 5–6 g/kg | 1–1.2 g/kg |

### 6.6 Algorithm Pseudocode

for muscle_group in program:
    target_sets = base_sets + week_offset
    if deload_week:
        target_sets *= 0.60
    readiness_mod = sleep_mod * hrv_mod * doms_mod * stress_mod
    target_sets *= readiness_mod
    target_sets = clamp(target_sets, 6, 22)

---

## 7. Data Schema

- `user_profile`
- `daily_metrics`
- `workout_log`
- `habit_master`
- `habit_log`

---

## 8. Key User Stories

- **US-01:** As a *Transitional* user I can toggle “life-event mode” so volume auto-drops 20%.
- **US-02:** As any user I can select preset habits or create a custom one.
- **US-03:** As any user I get a Coach reminder when skipping electrolytes ≥3 days.
- **US-04:** As any user I can share a 600×600 PNG of today’s calories, workout, and habit streak.
- **H-05:** If I miss Creatine 2 days consecutively, it disappears and I receive a tip.
- **H-06:** After a 7-day perfect habit streak, I can add a new habit.
- **H-07:** Share card shows only active habits.

---

## 9. Design Direction – Reference Screens


*Based on 10 reference UI screenshots: Spotify Home, Duolingo On‑Ramp, Philips Hue, Bing Mobile, Arc Search, How We Feel, Lifesum, Withings.*

### 9.1 Core Visual Themes
- **Dark Gradient “Stage Lighting”**  
  Radial or linear gradients keyed by Goal Mode (e.g., Beginner = teal→blue; Hard = magenta→violet).
- **Modular Card Grid**  
  2‑column card layout (Fuel, Iron, Recovery, Habits) with 8 px gap, 16 px corner radius, soft drop shadow.
- **Hero Illustration Onboarding**  
  Full‑bleed illustration + single headline + dual CTAs (“Try demo” / “Get started”).
- **Stepper Progress Bar**  
  Top‑strip progress with 4–5 dots, grey until active.
- **Bottom Tab Navigation**  
  Home / Log / Plan / Profile; center tab shows Habit modal progress indicator.
- **Metric Cards & Sparklines**  
  Inline micro‑charts (sparklines, streak dots) in the card headers.
- **Emotion/Habit Check‑in Modal**  
  Large icon glyph + friendly copy; gradient‑filled blob backdrop.

### 9.2 Design Tokens
- `radius-lg`: 16 px (cards, buttons)  
- `radius-xl`: 24 px (hero tiles, share card)  
- `spacing-unit`: 4 px  
- `primary-shade-beginner`: `#00C9A7 → #0068EF`  
- `primary-shade-normal`: `#00F260 → #0575E6`  
- `primary-shade-hard`: `#FF0080 → #7928CA`  
- `surface-dark`: `#121212`  
- `surface-light`: `#FFFFFF`  
- `shadow-04`: `0 2px 4px rgba(0,0,0,0.18)`

### 9.3 Motion & Micro‑Interactions
- Haptic “tick” on set completion (Spotify‑style click).  
- Fade‑out + toast animation when a habit is lost.  
- Parallax effect on onboarding hero gradients.

### 9.4 Typography
- **Headline**: Inter Bold, letter‑spacing `-0.5px`  
- **Body**: Inter Regular, 16 px font, 24 px line‑height  
- **Caption/Badge**: Inter Medium, 12 px font, 16 px line‑height

### 9.5 Accessibility & Themes
- WCAG 2.2 AA contrast compliance.  
- Support light/dark modes with token swaps.

### 9.6 Next Design Steps
1. Create a style‑tile (colors, type, shadows, motion).  
2. Build low‑fi wireframes (Home, Log, Plan, Habit modal, Share Card).  
3. Assemble clickable Figma prototype for alpha test.
