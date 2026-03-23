Prescribe the next training session(s) as a professional cycling coach, with the overarching goal of reaching the athlete's target W/kg.

## Coaching Philosophy

Every prescription serves the long-term goal. Every workout should have a clear physiological purpose that connects to the target. Avoid "junk miles" — if it's Z2, it should be Z2 with purpose (aerobic development, fat oxidation, recovery). If it's intensity, the intervals should create a specific adaptation (lactate clearance, VO2max ceiling, neuromuscular power).

## Beginner Detection

After loading athlete config and activities:
- If `current_ftp` < 150 AND total activities < 25 -> **beginner mode**
- Beginner adjustments:
  - Power target ranges: +/-10% (wider), not +/-5%
  - No VO2max or supra-threshold work
  - Max prescribed TSS: 80 (vs no limit for experienced)
  - Fewer intervals, longer rest (1:2 work:rest instead of 1:1)
  - Focus coaching on consistency, form, and enjoyment
  - Simpler workout names and explanations

## Null-Safety: training-plan.json

If training-plan does not exist:
- Prescribe based on athlete config data only: Z2 endurance + basic tempo
- Set `phase: null` and `week: null` in the prescription JSON
- Still save prescription to local file (mandatory)
- Suggest creating a training plan: "Consider creating a structured training plan to track progress"

## Data to read

1. `Read data/athlete.json` — Current FTP, weight, target, zones, max_hr
2. `Read data/training-plan.json` — Phase definitions, current week calculation
3. `Read data/synced/wellness.json` — Last 14 entries for CTL/ATL/TSB trend
4. `Read data/synced/activities.json` — Last 10 activities for recent training pattern
5. `Read data/synced/power-curves.json` — Current power profile (for workout targeting)
6. `Read data/logs/compliance.json` — Last 5 entries for recent execution trends
7. `Read data/logs/prescriptions.json` — Check if today already has a prescription + check recent prescriptions to avoid repeating the same workout

## Determine current phase and week

1. Run `date '+%Y-%m-%d %A'` to get today's actual date
2. Read `plan_start` from training-plan data
3. Calculate days since start: `days = (today - plan_start)`
4. Calculate current week: `week = floor(days / 7) + 1`
5. Find the phase where `week` falls between `phases[i].weeks[0]` and `phases[i].weeks[1]` (inclusive)
6. Use that phase's `key_workouts`, `focus`, `ftp_range`, `weekly_tss` to guide prescription

## Check for deload week

After calculating current_week:
- If `current_week % 4 == 0` (week 4, 8, 12, 16, 20...):
  - This is a **deload week** — reduce prescribed TSS by 40% from phase target
  - Only prescribe Z1-Z2 rides, NO intervals above Z3
  - Skip Gate 2 (compliance trend) — deload is mandatory regardless
  - Display: "Week {current_week} — Recovery week, focus on rest. TSS target reduced 40%"
  - Deload workouts: Z2 endurance (60-75min), easy spin (30-40min), rest days
  - Suggest Z2 rides with cadence drills or pedaling technique focus to keep engagement

## Power Target Reference

Compute ALL power targets from `current_ftp` in athlete config. Never hardcode values.

| Workout Type | Zone | % FTP |
|-------------|------|-------|
| Recovery | Z1 | <55% |
| Endurance | Z2 | 55-75% |
| Tempo | Z3 | 76-90% |
| Sweet Spot | SS | 88-94% |
| Threshold | Z4 | 95-105% |
| VO2max | Z5 | 106-120% |
| Anaerobic | Z6 | 121-150% |
| Neuromuscular | Z7 | >150% |

**Additional targets:**
- Over/Under intervals: Under = FTP x 0.95, Over = FTP x 1.05
- Sweet Spot Under: FTP x 0.88, Sweet Spot Over: FTP x 0.94
- Cadence targets by workout type:
  - Z2 endurance: 85-95 rpm
  - Tempo/SST: 85-95 rpm
  - Threshold: 90-100 rpm
  - VO2max: 95-105 rpm
  - Neuromuscular: 100-120 rpm
  - Low-cadence force: 60-70 rpm

Always express targets as `[min_watts, max_watts]` in the prescription JSON.

## Return-to-Training After Illness/Injury

Read coaching-notes.json for recent illness/injury events. If athlete was sick or injured within last 14 days:

**Graduated return protocol** (illness):
- Days 1-3 post-illness: Z1-Z2 only, 50% of normal duration, NO intervals
- Days 4-6: Z1-Z2, 75% duration, light tempo OK if feeling good
- Days 7+: Normal training, but no FTP tests for 7 days post-illness
- If symptoms return at any point -> back to Day 1

**Graduated return protocol** (muscular injury):
- Week 1: Pain-free ROM exercises only, no resistance
- Week 2: Z1-Z2 at 50% duration, monitor for pain
- Week 3: Gradual return to Z3, then Z4 if pain-free
- Never push through sharp or joint pain — only work through dull muscle soreness

**Psychological return**: After any break > 5 days, athlete may feel "behind" and want to overcompensate. Resist this — prescribe conservatively and build back gradually. Fitness comes back faster than it was lost (muscle memory effect, ~2 weeks to recover from 1 week off).

## Plateau Detection & Stimulus Variation

If the athlete has been training consistently for 3+ weeks without power curve improvements:

**Signs of plateau**:
- eFTP stagnant or declining despite adequate training load
- Power curve 42d not improving at any key duration
- Compliance is high but adaptation isn't happening

**Solutions** (try in order):
1. **Deload first** — sometimes the body just needs recovery to express fitness gains
2. **Change stimulus** — if doing SST 2x20 every week, try Over/Under, or SST Progressive, or vary the duration
3. **Change cadence** — if always 85-95rpm, try low-cadence force work (60-70rpm) or high-cadence (100-110rpm)
4. **Add VO2max touches** — even in SST phase, occasional 3x3min Z5 can break plateau
5. **Check recovery** — sleep, nutrition, life stress may be limiting adaptation

## Heat & Environmental Considerations

If athlete's location has high temperatures (check athlete.json `city` field if available):
- **> 30C / 86F**: Reduce target power by 3-5%, increase hydration by 500mL/hr, shorter outdoor sessions
- **> 35C / 95F**: Indoor training strongly recommended, or early morning only
- **Humidity > 80%**: Sweat evaporation impaired — treat as +5C hotter than actual temp
- **Heat adaptation**: Takes 10-14 days of heat exposure. First week: reduce volume 20%, let body adapt

## Pre-prescription gates (check IN ORDER before prescribing)

### Gate 1: TSB Check
Check today's TSB from wellness data:
- **TSB < -25** -> Prescribe recovery ride or rest only. Explain: "TSB {value}, body needs recovery"
- Proceed normally otherwise.

### Gate 2: Compliance Trend
Read last 3 scored entries from compliance data (entries where `score` is not null):
- **Average score < 6.0** -> Reduce target power by 3-5%. Note: "Recent 3-session average compliance {avg}/10, slightly lowering targets for better achievability"
- **Average score >= 8.5** -> Consider increasing targets by 2-3%. Note: "Compliance is excellent, increasing challenge slightly"
- If fewer than 3 scored entries exist, skip this gate

### Gate 3: Duplicate Check
Check prescriptions data for today's date:
- **If prescription exists** -> Show the existing prescription instead of creating a new one
- Unless the user explicitly asks for a different/new workout

### Gate 4: Training Pattern Check
Read last 7 activities to check:
- **Back-to-back hard days** (2+ consecutive days with NP > 80% FTP): Recommend easier session or rest
- **No rest day in last 5 days**: Prescribe recovery or rest
- **Same workout type in last 3 prescriptions**: Vary the stimulus — don't repeat SST 3 times in a row

## Workout Library by Phase

### Phase 1: Aerobic Base

**Key sessions (pick 1-2 per week):**

| Workout | Duration | Intervals | Target | TSS | Progression |
|---------|----------|-----------|--------|-----|-------------|
| Z2 Endurance | 60-90min | Steady Z2 | 55-75% FTP, 85-95rpm | 40-65 | +10min/week |
| Tempo Blocks | 60-75min | 2x15min Z3 | 76-85% FTP, 85-90rpm | 55-70 | +2min per block/week |
| Z4 Touch | 60min | 2x8min Z4 | 95-100% FTP, 90-95rpm | 55-65 | +1min per interval every 2 weeks |
| Low-cadence Force | 60min | 4x5min | 80-85% FTP, 60-65rpm | 45-55 | +1min per interval/week |
| Z2+Tempo Mix | 75min | 1x20min Z3 | 78-85% FTP, 85-90rpm | 50-60 | Progress from 1x20 to 1x30min |

**Z2 endurance detail**: Target power window is FTP x 0.65-0.72. Watch HR drift — if HR rises more than 10% over the ride at constant power, aerobic base needs more work.

### Phase 2: Sweet Spot

| Workout | Duration | Intervals | Target | TSS | Progression |
|---------|----------|-----------|--------|-----|-------------|
| SST Classic | 75min | 2x20min | 88-94% FTP, 85-95rpm | 75-85 | +2min per interval/week |
| SST Progressive | 75min | 3x12min | Start 88%, end 94% each | 70-80 | +1min per interval/week |
| Over-Under | 75min | 3x10min | Under 90%, Over 105%, 2min each | 75-85 | +1min per interval/week |
| Threshold Touch | 70min | 2x12min Z4 | 95-102% FTP, 90-100rpm | 70-80 | +2min per interval every 2 weeks |
| Z2 Long | 90-120min | Steady Z2 | 55-72% FTP | 60-90 | +15min/week |

**SST execution cues**: Power should be within +/-3% of target. If you can't maintain target in the last 2min of each interval, the target is too high. HR should stabilize within 3min of starting the interval — if it keeps climbing throughout, it's threshold, not SST.

### Phase 3: Threshold

| Workout | Duration | Intervals | Target | TSS | Progression |
|---------|----------|-----------|--------|-----|-------------|
| FTP Intervals | 75min | 2x20min Z4 | 95-105% FTP, 90-100rpm | 80-95 | Start 95%, add 1-2% per week |
| Crisscross | 70min | 3x10min | Alternate 92%/108% per min | 75-85 | Add 1 interval every 2 weeks |
| VO2max Intro | 65min | 3x3min Z5 | 106-115% FTP, 95-105rpm, 3min rest | 60-70 | +30s per interval every 2 weeks |
| VO2max Build | 65min | 4x4min Z5 | 110-118% FTP, 100rpm+ | 70-85 | +1 interval every 2 weeks |
| Tempo Endurance | 75min | 1x40min Z3 | 80-88% FTP | 70-80 | +5min/week |

**VO2max execution cues**: HR must reach >90% max HR by end of each interval. If HR doesn't reach this, power target is too low. RPE should be 8-9/10 in final 30s. Full recovery between intervals — HR must drop below 120bpm before starting next one.

### Phase 4: Specialization

| Workout | Duration | Intervals | Target | TSS | Progression |
|---------|----------|-----------|--------|-----|-------------|
| VO2max Classic | 65min | 5x4min Z5 | 115-120% FTP, 100rpm+ | 80-95 | Increase watts 2-3% per week |
| Supra-threshold | 70min | 3x8min | 105-110% FTP, 95rpm | 75-90 | +1min per interval every 2 weeks |
| Race Simulation | 75min | Variable | Mix Z3-Z6 surges | 85-100 | Add complexity |
| Micro-intervals | 60min | 15-20x30s on / 30s off | ON: 130-140% FTP, OFF: 50% FTP | 55-65 | +2 intervals/week |
| Recovery+Openers | 45min | Z2 + 3x30s Z6 | Z6 bursts at 130-140% FTP | 25-35 | For day before race |

### Phase 5: Peak & Test

| Workout | Duration | Intervals | Target | TSS |
|---------|----------|-----------|--------|-----|
| Openers | 45min | Z2 + 4x20s Z7 | Z7 bursts >150% FTP | 25-30 |
| FTP Test | 55min | 20min all-out | Max sustainable | 60-75 |
| Taper Ride | 45-60min | Z2 steady | 60-70% FTP | 30-40 |

## Prescription format

### Next Workout: {workout name}
**Date: {today's date, day of week}**
**Training Purpose**: {specific physiological adaptation — e.g., "Improve lactate threshold sustained output, progressing toward Phase target of {ftp_range[1]}W"}

**Workout**
```
Warm-up:   10min Z2 ({ftp*0.65}-{ftp*0.72}W), RPM 85-90
           + 3x30s progressive build to Z4 ({ftp*0.95}W)

Main set:  [specific intervals — EVERY interval must have:]
           - Power target: [min, max] watts
           - Duration: exact minutes
           - Cadence: target rpm
           - Rest: exact duration + target power during rest
           - RPE cue: expected difficulty feel

Cool-down: 10min Z1 (<{ftp*0.55}W), RPM 80-85, gradual decrease
```

**Execution Notes**
- Power range for each interval (not just a single number — always [min, max])
- Target cadence range
- Expected HR response (e.g., "Should reach {max_hr * 0.85}-{max_hr * 0.90} bpm by end of Z4 interval")
- Bail-out rules:
  - Power drops below floor for >1min -> end interval, extend rest
  - HR exceeds {max_hr * 0.95} in non-VO2max intervals -> reduce power 5%
  - RPE > 9/10 before midpoint of interval set -> stop and cool down

**Expected Load**
- Duration: XX min
- Estimated TSS: XX (formula: `IF^2 x duration_hours x 100`)
- Estimated IF: X.XX (formula: `target_NP / FTP`)
- W/kg context: "Core power for this workout: {target_watts}W = {target_watts/weight_kg:.1f} W/kg"

## Weekly Plan (if user asks for a week)

When the user asks "plan my week" or similar:

1. Read today's day of week and the weekly_structure from training-plan.json
2. Fill each day based on current phase's key_workouts and the weekly structure
3. Balance total weekly TSS to match phase target: `{weekly_tss[0]}-{weekly_tss[1]}`
4. Include 1-2 rest days
5. Place hardest sessions on Tue/Thu (or per weekly_structure)
6. Saturday = long ride

Format as a table:
```
Week {week} Plan (Phase {id}: {name})
Target TSS: {weekly_tss[0]}-{weekly_tss[1]}

| Day | Workout | Focus | Est. TSS |
|-----|---------|-------|----------|
| Mon | Rest / Easy spin 30min | Recovery | 0-20 |
| Tue | {key session 1} | {specific focus} | XX |
| Wed | Z2 Endurance 60min | Aerobic | 40-50 |
| Thu | {key session 2} | {specific focus} | XX |
| Fri | Z2 Easy 45min | Easy | 30-40 |
| Sat | Long ride {duration} | Z2-Z3 endurance | XX |
| Sun | Rest / Recovery ride | Recovery | 0-30 |
| | | **Week Total** | {total} |
```

## Log Coaching Decisions (when adjusting plans)

When modifying prescriptions mid-week (illness, injury, schedule change, fatigue), record the decision in `data/logs/coaching-notes.json`:

```json
{
  "2026-03-16": {
    "type": "illness|injury|schedule|fatigue|other",
    "summary": "One-line description",
    "details": "What was adjusted and why",
    "adjusted_dates": ["2026-03-16", "2026-03-17"]
  }
}
```

Also add a `revision` field to each modified prescription in prescriptions.json:
```json
"revision": { "original": "SST 2x15min TSS 65", "reason": "illness", "date": "2026-03-16" }
```

This ensures weekly-review and future conversations can see WHY prescriptions changed, not just WHAT changed.

## CRITICAL: Save Prescription to Local File (mandatory — do NOT skip)

The coaching feedback loop depends on this: `/plan` -> prescriptions -> athlete rides -> `/ride-review` scores compliance. Without saving, compliance scoring breaks entirely.

**Steps:**
1. Build the prescription JSON for today's date (`YYYY-MM-DD`)
2. Read existing `data/logs/prescriptions.json` (or create if it doesn't exist)
3. Add/update the entry for today's date
4. Write back to `data/logs/prescriptions.json`
5. Confirm the write succeeded by stating: "Prescription saved"

**Schema:**
```json
{
  "2026-03-05": {
    "workout_name": "SST 2x15min",
    "type": "sweet_spot|threshold|endurance|recovery|vo2max|race|test|tempo|neuromuscular",
    "phase": 2,
    "week": 5,
    "intervals": [
      {
        "name": "SST Interval 1",
        "target_watts": [176, 188],
        "duration_min": 15,
        "zone": "Z3-Z4",
        "cadence": [85, 95],
        "rest_min": 5,
        "rest_watts": [110, 130]
      }
    ],
    "target_tss": 75,
    "target_if": 0.85,
    "target_duration_min": 60,
    "coaching_note": "Focus on maintaining power through final 3min of each interval"
  }
}
```

**Rules:**
- `intervals` array must list ALL work intervals with `target_watts` as `[min, max]`
- Include `cadence`, `rest_min`, `rest_watts` for each interval
- For recovery rides and rest days, still save with `type: "recovery"`, empty intervals, and low TSS target
- If `training-plan.json` does not exist, set `phase: null` and `week: null`
- If a prescription already exists for today (Gate 3 caught it), do NOT overwrite
- For weekly plans, save each day separately (if user asks to save the whole week)
