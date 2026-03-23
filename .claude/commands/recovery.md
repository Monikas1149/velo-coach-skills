Provide personalized recovery and nutrition advice based on today's training load, fitness status, and training phase. Recovery is half the equation — you can't adapt if you don't recover.

## Beginner Detection

After loading athlete config:
- If `current_ftp` < 150 -> **beginner mode**
- Beginner adjustments:
  - Protein: 1.2 g/kg (general active adult) instead of phase-based calculation
  - Simpler meal suggestions (avoid overly specific supplement recommendations)
  - Recovery focus on: sleep, basic stretching, adequate protein — skip advanced modalities
  - TSB recovery threshold: < -15 = AGGRESSIVE (not -20)

## Null-Safety: training-plan.json

If training-plan does not exist OR `plan_start` is null:
- Skip Phase-specific Nutrition Notes (Step 4)
- Use general nutrition guidance instead of phase-specific advice
- Protein defaults to 1.2 g/kg

## Data to read

1. `Read data/athlete.json` — weight_kg, current_ftp, max_hr, target_ftp
2. `Read data/training-plan.json` — Current phase (calculate week from plan_start)
3. `Read data/synced/wellness.json` — Last 7 entries for CTL/ATL/TSB trend + today's TSS
4. `Read data/synced/activities.json` — Today's activities (TSS, duration, NP, kJ)
5. `Read data/synced/laps/{activity_id}.json` — Today's lap data for NP/IF calculation
6. `Read data/logs/prescriptions.json` — Tomorrow's prescription (affects recovery priority)

## Step 1: Determine Day Type

Get today's activity data and compute:

**IF (Intensity Factor)** = NP / FTP
- Use `true_np` from lap file if available, otherwise `weighted_average_watts` from activities.json
- If no NP available, estimate from average_watts x 1.05

**TSS** = IF^2 x duration_hours x 100
- Use TSS from activities.json if available, otherwise compute from IF

Classify:

| Day Type | Condition | Carb Target (g/kg) |
|----------|-----------|-------------------|
| REST | No training, TSS = 0 | 3-4 |
| EASY | TSS < 50 AND IF < 0.65 | 4-5 |
| MODERATE | TSS 50-100 OR IF 0.65-0.80 | 5-7 |
| HARD | TSS 100-180 OR IF 0.80-0.95 | 7-8 |
| MASSIVE | TSS > 180 OR duration > 3hr | 8-10 |
| RACE | Activity name contains "Race", "Crit", "Time Trial" | 8-10 |

If multiple activities today, sum their TSS and use the highest IF for classification.

## Step 2: Determine Recovery Level

Combine day_type with TSB to set recovery urgency:

| Level | Trigger | Color |
|-------|---------|-------|
| LIGHT | REST/EASY day AND TSB > -10 | Green |
| STANDARD | MODERATE day OR TSB between -10 and -20 | Yellow |
| AGGRESSIVE | HARD day OR TSB between -20 and -30 | Orange |
| EMERGENCY | MASSIVE day OR TSB < -30 | Red |

**Upgrade rules** (apply in order, each can upgrade one tier):
- TSB < -30 -> upgrade one tier
- Tomorrow's prescription is high-intensity (type = threshold/vo2max/race) -> upgrade one tier
- 3+ consecutive training days (no rest in last 3 days) -> upgrade one tier
- Max level is EMERGENCY

## Step 3: Calculate Macros

Read `weight_kg` from athlete config (W = weight).

**Protein** (daily requirement, varies by phase and day type):

| Phase | Base (g/kg) | Hard/Massive day bonus |
|-------|-------------|----------------------|
| Phase 1-2 | 1.4 | +0.2 |
| Phase 3-4 | 1.6 | +0.2 |
| Phase 5 | 1.6 | +0.2 |
| No plan | 1.2 | +0.2 |

Example: Phase 1, 70kg, HARD day -> 70 x (1.4 + 0.2) = 112g protein

**Carbs** (varies by day_type from Step 1):
- Use midpoint of the g/kg range from the Day Type table
- Show the range as context
- Example: MODERATE day, 70kg -> 70 x 6.0 = 420g (range: 350-490g)

**Fat** (minimum floor):
- Floor: W x 0.8 g/kg (do not go below this)
- This is a minimum, not a target — fill remaining calories after protein + carbs

**Total calories estimate**:
```
calories = protein x 4 + carbs x 4 + fat x 9
```

**Hydration**:
```
base = weight_kg x 35 mL (resting requirement)
training = ride_duration_hours x 700 mL (average sweat loss)
total = base + training

# Round to nearest 0.5L for practical advice
```
- Hot weather / high intensity: add 200-300 mL per hour
- Post-ride: drink 1.5x fluid lost (weigh before/after if possible)

**Electrolyte needs** (for rides > 60min or HARD+ days):
- Sodium: 500-1000mg per hour of riding
- Practical: 1 electrolyte tab per 500mL during/after ride
- Add pinch of salt to recovery meal if no electrolyte supplement

## Step 4: Phase-specific Nutrition Notes

Read current phase from training-plan data:

- **Phase 1 (Aerobic Base)**: "Train low" strategy permitted for Z2 sessions — riding fasted or with minimal carbs is OK for rides < 90min. Z4+ sessions MUST be fully fueled. Morning fasted Z2 can improve fat oxidation.
- **Phase 2 (Sweet Spot)**: Full fueling required for all sessions. Pre-ride meal 2-3hr before. Intra-ride carbs for sessions > 60min (30-40g/hr).
- **Phase 3 (Threshold)**: Full fueling + practice race-day nutrition. High-intensity days: target upper end of carb range. Start training gut for higher carb intake during rides (60g/hr target).
- **Phase 4 (Specialization)**: Race nutrition rehearsal. Practice 60-90g/hr carb intake during long intensity sessions. High-carb days on key session days (8-10 g/kg).
- **Phase 5 (Peak)**: Taper nutrition: reduce total calories ~10% as volume drops, maintain protein. Carb loading protocol for final test: 3 days at 10-12 g/kg, reduce fiber, minimize residue.

## Step 5: Intra-ride Fueling Guide

Based on today's ride duration or tomorrow's planned duration:

| Duration | Carbs/hr | Hydration/hr | Practical Guide |
|----------|----------|-------------|-----------------|
| < 60min | Not required | 500mL water | Z2 fasted OK (Phase 1 only) |
| 60-90min | 30-40g | 500-750mL | 1 gel every 30min OR half energy bar |
| 90-150min | 40-60g | 750mL | 1 gel or drink mix every 20-25min |
| > 150min | 60-90g | 750-1000mL | Dual-source (2:1 glucose:fructose). Alternate gels + bar + drink mix |

**Pre-ride meal timing** (for key sessions):
- 3hr before: Full meal (carbs + protein + fat). Example: rice + egg + banana
- 2hr before: Moderate meal (carbs + protein). Example: oatmeal + whey
- 1hr before: Light snack (simple carbs). Example: banana, rice cake, gel
- 30min before: Gel or sports drink only if needed

## Report Format

```
### Recovery Protocol — {date}

**Today's Load**: {day_type}
- TSS: {total_tss} | Duration: {duration} | IF: {if_value}
- NP: {np}W ({np/weight:.1f} W/kg) | kJ: {kj}
**Recovery Level**: {level_name}
**TSB Trend**: {tsb_today} -> estimated tomorrow {tsb_tomorrow_est}

---

### Nutrition Targets

| Nutrient | Target | g/kg | Notes |
|----------|--------|------|-------|
| Carbs | {carbs}g | {carbs_gpkg} | {day_type} day |
| Protein | {protein}g | {protein_gpkg} | Phase {id} |
| Fat | >={fat}g | >={fat_gpkg} | Minimum intake |
| Water | {hydration}L | {ml_per_kg} ml/kg | Including training losses |
| Calories | ~{calories} kcal | | Estimate |
```

### Nutrient Timing

**If rode today** (prioritize post-ride recovery window):
```
IMMEDIATE (0-30min post-ride — recovery window):
   -> Protein 25-30g + Carbs 40-60g + Electrolytes
   -> Practical: whey shake + banana + 500mL electrolyte drink
   -> Purpose: muscle repair initiation + glycogen replenishment start

1-2hr post-ride (first real meal):
   -> Full meal, carb + protein focused
   -> Practical: rice/pasta + chicken/fish/eggs + vegetables
   -> Protein 30-40g in this meal

3-4hr post-ride (second meal/snack):
   -> Continue carb and protein replenishment
   -> Practical: sandwich/rice ball + milk/yogurt

Before bed (1hr before):
   -> Casein protein or milk (slow-release, overnight repair)
   -> Optional: Magnesium glycinate 200-400mg (sleep quality + muscle relaxation)
   -> Avoid: caffeine, large fluid volumes (disrupts sleep)
```

**If rest day**:
```
Normal 3 meals: distribute protein evenly (~30g/meal)
Moderate carbs: whole grains + fruits/vegetables, no extra supplementation needed
Hydration: steady intake, target {base_hydration}L
```

### Physical Recovery Protocol

**Based on recovery level — select ONE matching tier:**

---

#### LIGHT (REST/EASY + TSB > -10)

**Active Recovery**: 15-20min light activity (walking, very easy spin <50% FTP)
**Foam Roll**: 2-3 muscle groups, 30-45 seconds each
**Stretching**: Full body light stretching 10min, hold each position 20-30 seconds
**Sleep**: 7-8hr, normal schedule

---

#### STANDARD (MODERATE day or TSB -10 to -20)

**Active Recovery**: 20-30min light activity (walking, Z1 spin <55% FTP)
**Foam Roll** (15min) — focus muscle groups, 60 seconds each, pause on tender points:
- Quadriceps — primary cycling muscles
- Hamstrings — pull phase
- IT Band (outer thigh) — repetitive pedaling friction
- Glutes — hip extension power
- Calves — ankle stabilization

**Stretching** (10-15min) — static stretching, hold each 30 seconds:
- Hip flexor stretch (half-kneeling)
- Seated forward fold (hamstrings)
- Quad stretch (side-lying)
- Calf stretch (wall push)
- Thoracic rotation (open book)

**Relaxation**: Hot shower/bath 10-15min
**Sleep**: Target 8hr

---

#### AGGRESSIVE (HARD day or TSB -20 to -30)

**Active Recovery**: 20-30min very light activity, completely effortless
**Foam Roll** (15-20min) — all key muscle groups, 60-90 seconds each:
- Quadriceps — front + outer
- Hamstrings
- IT Band — slow rolling
- Glutes — including piriformis (seated on ball)
- Calves — gastrocnemius + soleus
- Upper back/thoracic — improve riding-posture stiffness
- TFL (outer hip) — hip flexor connection

**Stretching** (15min) — static focus, hold 30-45 seconds each:
- Hip flexor stretch (half-kneeling, push hips forward)
- Pigeon pose — deep glute + piriformis
- Seated forward fold — hamstrings
- Quad stretch — side-lying
- Calf stretch — wall push
- Thoracic rotation — open book
- Neck side bend stretch

**Relaxation methods**:
- Compression clothing for 2-4hr
- Hot shower/bath 15-20min
- Breathing exercises 5min — 4-7-8 method (inhale 4s -> hold 7s -> exhale 8s)
- Consider contrast shower (hot 2min -> cold 30s x 3 cycles)

**Sleep**: Target 8.5hr+, in bed by 22:00, reduce blue light 1hr before bed

---

#### EMERGENCY (MASSIVE day or TSB < -30)

**Recovery is the ONLY priority today — do not schedule any training**

**Active Recovery**: Light walking only 15-20min, no cycling
**Foam Roll** (20min) — full body release, 90 seconds+ per group:
- All AGGRESSIVE-level muscle groups
- Add: quadratus lumborum (side-lying), plantar fascia (foot on ball)

**Stretching** (15-20min) — relaxation-oriented:
- All AGGRESSIVE-level stretches
- Add: Child's pose — full body relaxation
- Add: Supine spinal twist — lower back release
- Add: Legs up the wall 5-10min — promote leg blood return

**Full recovery protocol**:
- Compression clothing for 3-4hr
- Cold bath/shower 10-15min @ 10-15C, or contrast shower
- Breathing exercises 10min — box breathing (inhale 4s -> hold 4s -> exhale 4s -> hold 4s)
- Afternoon power nap 20min (complete before 14:00)
- Electrolyte supplementation (sodium + potassium + magnesium)

**Sleep**: 9hr+
- Pre-bed magnesium (Mag glycinate 200-400mg)
- Pre-bed tart cherry juice 30mL (natural melatonin + anti-inflammatory)
- Room temperature 18-20C, full blackout

**Next day**: No high intensity, full rest day or Z1 only
**Monitor**: If TSB still < -25 next day -> consecutive rest, do not force training

---

### Sleep Optimization (Evidence-based)

Sleep is the #1 recovery tool. Consistently good sleep is non-negotiable for performance goals:

**Core principles**:
- 7.5-9hr per night (8hr minimum for hard training blocks)
- Consistent bed/wake times (+/-30min)
- No caffeine after 14:00
- No screens 30min before bed (or use blue light filter)
- Room: dark, cool (18-20C), quiet

**During high-intensity training blocks**:
- Add 30-60min total sleep (earlier bedtime)
- If night sleep < 7hr, add 20min afternoon nap (before 15:00)
- Track sleep quality — if trend is declining, reduce training load

### Tomorrow Preview

```
{if tomorrow has prescription}
> Tomorrow: {workout_name} ({type})
> Expected TSS: {target_tss}, Duration: {target_duration}min
> Pre-ride preparation:
>   - Meal: {pre-ride meal based on workout intensity}
>   - On-bike fueling: {intra-ride nutrition if applicable}
>   - Warm-up: {warm-up protocol based on workout type}
{else}
> No prescription for tomorrow. Based on TSB ({tsb_est}) and recovery status, decide then.
> Suggestion: {rest recommendation based on recovery level}
{end if}
```

## Special Cases

### Double session day
- Sum all TSS for day_type classification
- Protein: add +0.2 g/kg to target
- Recovery window: prioritize first session's recovery in the gap between sessions
- Second recovery window after last session of the day

### Weight training day
- Protein timing: 30g within 1hr post-weights
- If combined with cycling: eat between sessions (carbs + protein)
- Extra attention to: quad/glute/hamstring foam rolling
- Note: "Protein timing is important on weight training days — supplement within 1hr post-session"

## Immune System & Illness Prevention

Hard training creates a 3-72hr "open window" of immune suppression. This is especially relevant during high-load weeks:

**Immune risk factors** (check and warn):
- TSS > 150 in a single session -> "WARNING: Post-high-load immune window — avoid crowds, stay warm, increase vitamin C intake"
- 3+ consecutive hard days -> elevated cortisol, immune suppression risk
- Sleep < 7hr on a training day -> immune function drops ~30%

**Prevention protocol after very hard sessions**:
- Avoid crowds/confined spaces for 2-3hr post-training
- Vitamin C: 200-500mg post-ride (reduces upper respiratory infection risk by ~50% in athletes, Hemila 2013)
- Zinc: 15mg/day during high-load blocks
- Glutamine: 5g post-exercise (supports gut barrier and immune cells)
- Avoid fasting after hard sessions — carb intake post-exercise restores immune function

**If athlete reports feeling sick** (sore throat, runny nose, fatigue):
- **Neck check**: symptoms above neck (runny nose, mild sore throat) -> Z1-Z2 only, shortened duration
- **Neck check**: symptoms below neck (chest congestion, body aches, fever) -> COMPLETE REST, no training
- After illness: return at 50% volume/intensity for first 3 days, then 75% for 3 days, then normal
- Do NOT do FTP tests within 7 days of illness recovery

## Gut Health & Fueling Tolerance

Individual carb tolerance varies widely. Help the athlete train their gut:

**Gut training protocol** (Phase 2-3, preparing for race-day nutrition):
- Week 1-2: 30g/hr carbs during rides > 60min
- Week 3-4: 40-50g/hr
- Week 5-6: 60g/hr (target for most athletes)
- Week 7+: 60-90g/hr if tolerated (dual-source glucose:fructose 2:1)
- Signs of GI distress: bloating, cramps, nausea -> back off 10-15g/hr, try different products

**Practical fueling tips**:
- Liquid carbs (drink mix) are easier to absorb than solid (bars) during high intensity
- Avoid high-fiber, high-fat foods within 2hr of training
- Caffeine (3-6mg/kg, 45-60min pre-ride) improves performance ~3%, but can cause GI issues — test in training first
- If athlete reports stomach issues: check if eating too close to ride, or wrong carb source

## Supplement Guide (Evidence-Ranked)

Only recommend supplements with strong evidence. Most supplements are a waste of money.

**Tier 1 — Strong evidence (recommend):**
| Supplement | Dose | Timing | Benefit |
|-----------|------|--------|---------|
| Caffeine | 3-6mg/kg | 45-60min pre-ride | ~3% performance boost, RPE reduction |
| Creatine monohydrate | 3-5g/day | Any time (daily) | Supports high-intensity repeats, muscle recovery |
| Vitamin D | 1000-2000 IU/day | With meal | Most athletes are deficient; supports immune + bone |
| Magnesium glycinate | 200-400mg | Before bed | Sleep quality, muscle relaxation, cramp prevention |

**Tier 2 — Moderate evidence (consider during hard blocks):**
| Supplement | Dose | Timing | Benefit |
|-----------|------|--------|---------|
| Beta-alanine | 3-6g/day (split doses) | Daily | Buffers muscle acidity; helps 1-4min efforts |
| Tart cherry juice | 30-60mL concentrate | 2-3hr pre-sleep | Anti-inflammatory, natural melatonin, DOMS reduction |
| Omega-3 (fish oil) | 2-3g EPA+DHA/day | With meal | Anti-inflammatory, joint health |
| Zinc | 15mg/day | With meal (not with calcium) | Immune support during high-load blocks |

**Tier 3 — Limited evidence (don't waste money):**
BCAAs (redundant if protein adequate), glutamine (marginal), collagen (insufficient evidence for joints), HMB (only useful if untrained)

## Cold Therapy — When to Use and When to Avoid

Cold therapy (ice baths, cold showers) is nuanced — it can HELP or HARM depending on timing:

**USE cold therapy** (within 0-2hr post-exercise):
- After races or competitions (recovery > adaptation)
- After massive TSS days (> 200 TSS, need rapid recovery for next day)
- During multi-day events or race blocks

**AVOID cold therapy** (can blunt training adaptation):
- After regular training sessions in base/build phases — you WANT inflammation for adaptation
- After strength training — cold impairs muscle hypertrophy signals
- Within 24hr of a key session where you want adaptation

**Protocol if using**: 10-15min immersion at 10-15C. Contrast (hot 2min -> cold 30s x 3-4 cycles) is a gentler alternative with similar perceived benefits.

## Recovery Evidence Hierarchy

When explaining recovery recommendations, prioritize by evidence level:
1. **Sleep** — strongest evidence, largest effect
2. **Nutrition** (protein + carb timing) — strong evidence
3. **Active recovery** — moderate evidence
4. **Foam rolling** — moderate evidence for DOMS reduction
5. **Stretching** — moderate evidence for ROM maintenance
6. **Compression** — limited evidence, mainly psychological
7. **Cold/heat therapy** — context-dependent (cold may blunt adaptation if used routinely)
8. **Supplements** — limited evidence (except: caffeine, creatine, Mg, Vitamin D)
