Detailed post-workout analysis as a professional cycling coach. Every ride is a data point toward the long-term goal — connect the analysis back to the athlete's target.

## Beginner Detection

After loading athlete config and activities:
- If `current_ftp` < 150 AND total activities < 25 -> **beginner mode**
- Beginner adjustments:
  - Compliance scoring: weight `duration_complete` at 40% (not 25%) and `power_accuracy` at 25% (not 40%)
  - Use simpler language; celebrate completions, don't over-critique
  - Focus coaching notes on form, consistency, and enjoyment

## Data Format Handling

Activities may come from different sources with different field availability:

**intervals.icu-sourced activities**:
- May have `average_watts: null` — use `weighted_average_watts` as fallback
- Laps may have `type` field (WORK, RECOVERY, HARD, THRESHOLD, etc.) AND `zone` field
- Identify work intervals by: `type` is WORK/HARD/THRESHOLD/VO2MAX (preferred) or watts > 55% FTP

**Strava-sourced activities**:
- Have `average_watts` AND `weighted_average_watts`
- Laps have `name` (e.g., "Lap 1") but NO `type` field
- Identify work intervals by: watts > 55% FTP AND duration > 30s

Always check which fields exist and use the best available.

## Data to read

1. `Read data/athlete.json` — Get current FTP, zones, weight_kg, max_hr, target_ftp for calculations
2. `Read data/training-plan.json` — Determine current phase and expected workout types
3. `Read data/synced/activities.json` — Find the most recent activity (or activities from the last session date)
4. `Read data/synced/laps/{activity_id}.json` — Read lap data for each activity to review. Use `true_np` field (calculated from power stream) instead of `weighted_average_watts` from activities — it is more accurate
5. `Read data/synced/wellness.json` — Get the TSB for context on freshness (last few entries)
6. `Read data/logs/prescriptions.json` — Look up the prescription for this activity's date (also check day before). **If the prescription has a `revision` field, use the revised version** (the original was changed due to illness/schedule/etc.)
7. `Read data/logs/coaching-notes.json` — Check if there are coaching notes for this date (illness, injury, plan adjustment) that should contextualize the compliance assessment

If the user specifies a date, review that date's activities instead.

## Zone calculation

Read `current_ftp` and `zones` from athlete config. Compute zone boundaries in watts dynamically:

```
For each zone Z1-Z7:
  watts_min = current_ftp x zones.ZN[0]
  watts_max = current_ftp x zones.ZN[1]
```

Always read from athlete config — never hardcode zone percentages.

## Analysis format

### Ride Review: {date} — {activity name}

**Overview**
| Metric | Value | Notes |
|--------|-------|-------|
| Duration | XX:XX | |
| NP | XXX W | {NP/weight:.1f} W/kg |
| Avg Power | XXX W | |
| Avg HR | XXX bpm | {avg_hr/max_hr*100:.0f}% MHR |
| Max HR | XXX bpm | {max_hr_actual/max_hr*100:.0f}% MHR |
| kJ | XXXX | |
| IF | X.XX | NP / FTP = {NP}/{FTP} |
| VI | X.XX | NP / Avg Power (see interpretation below) |
| TSS | XXX | IF^2 x hours x 100 |

**IF (Intensity Factor)** interpretation:
- < 0.75: Recovery/Z2
- 0.75-0.85: Tempo/SST
- 0.85-0.95: Threshold race effort
- 0.95-1.05: TT effort / hard race
- > 1.05: Short intense effort or FTP might be set too low

**VI (Variability Index)** interpretation:
- 1.00-1.05: Very steady (time trial, trainer)
- 1.05-1.10: Moderately variable (structured intervals)
- 1.10-1.20: Variable (group ride, road race)
- > 1.20: Very variable (crit, race with surges)

### Interval Analysis

For each work interval (filter criteria below):

```
Interval 1: {duration} @ {actual_watts}W ({%FTP}% FTP, {watts/weight:.1f} W/kg)
|-- Target: {prescribed_min}-{prescribed_max}W -> {assessment}
|-- HR: avg {hr}bpm ({%MHR}%), max {max_hr}bpm
|-- Cadence: {cadence}rpm
|-- Note: {interval-specific coaching note}
```

Filter work intervals:
- If laps have `type` field: keep WORK/HARD/THRESHOLD/VO2MAX, exclude RECOVERY/REST
- If laps don't have `type`: exclude laps with watts < 55% FTP (FTP x 0.55) or duration < 30s

### Cardiac Drift & Fatigue Analysis

**Cardiac drift** measures aerobic efficiency — how much HR rises over time at constant power. Calculate for rides > 30min:

```
cardiac_drift = (avg_HR_second_half - avg_HR_first_half) / avg_HR_first_half x 100
```

If laps don't split naturally into halves, use the first and last work intervals:
- Compare HR of first work interval vs last work interval at similar power

Interpretation:
- < 3%: Excellent aerobic fitness, well within capacity
- 3-5%: Normal, good endurance
- 5-8%: Moderate fatigue, approaching limits
- > 8%: Significant fatigue — intensity may be too high for current fitness, or insufficient fueling

For Z2 endurance rides:
- Drift < 5% = good aerobic base indicator
- Drift > 5% = aerobic base needs more work

**Efficiency Factor (EF)** = NP / avg_HR
- Track trend over time: EF increasing at same intensity = improved fitness
- "EF: {np/avg_hr:.2f} — {comparison to recent similar rides}"
- Compare EF across similar workout types (Z2 vs Z2, SST vs SST) — EF is only meaningful when comparing same intensity band
- EF improving at same power = aerobic adaptation happening (heart pumping more blood per beat)
- EF declining at same power = possible fatigue accumulation, dehydration, heat, or overtraining

**Cadence & Fatigue Patterns**
- Compare cadence of first vs last work intervals:
  - Cadence drop > 5rpm: "WARNING: Cadence dropped {delta}rpm in later intervals — possible muscle fatigue, consider building base cadence or adding single-leg drills"
  - Cadence stable: "Cadence stable — good muscular endurance"
- Optimal cadence ranges by zone:
  - Z2 endurance: 85-95rpm (higher cadence reduces muscular strain)
  - SST/Threshold: 85-100rpm (personal preference, but consistency matters)
  - VO2max intervals: 95-105rpm (higher cadence shifts load to cardiovascular system)
  - Sprints: 100-120rpm (neuromuscular recruitment)
- If avg cadence < 80rpm in Z2: "TIP: Z2 cadence is low ({cadence}rpm) — try raising to 85-95rpm to reduce muscle damage and improve recovery"

### Indoor vs Outdoor Considerations

When activity source is Zwift, MyWhoosh, or trainer:
- **Power accuracy**: Smart trainers typically read 1-5% higher than outdoor power meters. Note this when comparing indoor vs outdoor performances
- **RPE calibration**: Indoor rides feel harder at same power due to heat buildup, no wind cooling, and psychological factors. An IF of 0.85 indoors may feel like 0.90 outdoors
- **HR response**: Expect HR 5-10bpm higher indoors at same power (heat, no evaporative cooling). Adjust cardiac drift interpretation accordingly — 5% drift indoors is equivalent to ~3% outdoors
- **Variability Index**: Indoor structured workouts should have VI 1.00-1.05. If VI > 1.10 on a trainer, rider may be struggling to hold steady power — suggest using ERG mode or focusing on smoother pedaling
- **Don't penalize**: Don't dock compliance for indoor-outdoor power differences

When activity is outdoors:
- **Terrain effects**: Hilly courses naturally produce higher NP relative to avg power (higher VI). This doesn't indicate poor execution
- **Wind/drafting**: Group rides with drafting show lower NP for same RPE
- **Stopping time**: Outdoor rides with traffic lights/stops have lower moving time %. Compliance scoring should use moving_time, not elapsed_time

### Power Distribution

If enough data, show time/power distribution:
```
Zone Distribution:
Z1 (<{z1_max}W):  [======    ]  12min (20%)
Z2 ({z2_range}W): [========  ]  24min (40%)
Z3 ({z3_range}W): [====      ]  10min (17%)
Z4 ({z4_range}W): [==        ]   8min (13%)
Z5+ (>{z5_min}W): [=         ]   6min (10%)
```

## Compliance Scoring

### Case 1: Prescription exists

Look up the prescription for the activity's date. If not found, also check the day before (user might ride a prescribed workout a day late).

**Step 1: Match prescribed intervals to actual laps**

Filter actual laps to work intervals only (criteria above). Match by sequence order: 1st prescribed interval -> 1st work lap, 2nd -> 2nd, etc. If counts differ, match as many as possible and note the mismatch.

**Step 2: Calculate 4 factors** (each 0.0 to 1.0)

**power_accuracy** (weight: 40%)
For each matched interval pair:
```python
target_mid = (prescribed_max + prescribed_min) / 2
target_range = (prescribed_max - prescribed_min) / 2
deviation = abs(actual_watts - target_mid)

if deviation <= target_range:
    # Within prescribed range
    score = 1.0
elif deviation <= target_range + target_mid * 0.05:
    # Within 5% of range boundary
    score = 0.85
elif deviation <= target_range + target_mid * 0.10:
    # Within 10% of range boundary
    score = 0.70
else:
    # Beyond 10% — linear scale down, minimum 0.30
    overshoot = (deviation - target_range) / target_mid
    score = max(0.30, 1.0 - overshoot * 3)
```
Average across all matched intervals.

**duration_complete** (weight: 25%)
```python
score = min(1.0, actual_total_moving_time_sec / (prescribed_target_duration_min * 60))
# Minimum 0.30 for very short sessions
score = max(0.30, score)
```

**interval_consistency** (weight: 20%)
```python
# For work intervals only (>=2 intervals required)
powers = [interval.watts for interval in work_intervals]
cv = stdev(powers) / mean(powers)  # coefficient of variation
score = max(0.30, min(1.0, 1.0 - cv * 3))
```
If only 1 work interval, score = 0.85 (can't measure consistency with 1 data point).

**hr_appropriateness** (weight: 15%)
```python
max_hr = athlete.max_hr  # from athlete.json

for Z2 work (watts < 75% FTP):
    target_hr_ceiling = max_hr * 0.75
    if avg_hr <= target_hr_ceiling:
        score = 1.0
    else:
        # Each 2% over = -0.10
        pct_over = (avg_hr - target_hr_ceiling) / max_hr * 100
        score = max(0.30, 1.0 - (pct_over / 2) * 0.10)

for Z4+ work (watts >= 90% FTP):
    target_hr_floor = max_hr * 0.85
    if avg_hr >= target_hr_floor:
        score = 1.0
    else:
        # Each 3% under = -0.10
        pct_under = (target_hr_floor - avg_hr) / max_hr * 100
        score = max(0.30, 1.0 - (pct_under / 3) * 0.10)

for Z3 work (75-90% FTP):
    # HR should be 75-85% MHR
    if max_hr * 0.75 <= avg_hr <= max_hr * 0.85:
        score = 1.0
    else:
        score = 0.80  # slightly off but acceptable
```

**Composite score**: `(power x 0.40 + duration x 0.25 + consistency x 0.20 + hr x 0.15) x 10`

Display as:
```
Compliance: 7.5 / 10
|-- Power Accuracy:       85% [=================   ] (0.85)
|-- Duration Complete:   100% [====================] (1.00)
|-- Interval Consistency: 88% [=================   ] (0.88)
|-- HR Appropriateness:   90% [==================  ] (0.90)
```

### Case 2: Race or Group Ride (no compliance scoring)

If activity name contains "Race", "Group Ride", "Pacer Group", "Crit", "Time Trial":
- Set score = null, factors = null
- Provide **race-specific analysis** instead:
  - Average power, NP, W/kg
  - Surge count: how many times power exceeded 120% FTP for >10s
  - Max power and sprint analysis (if max power > 150% FTP)
  - Pacing strategy: did power fade in second half? (negative split = good tactics)
  - Position (if visible from activity name)
- Note: "Races/group rides are not scored for compliance"

### Case 3: No prescription found

Compute factors that don't require a prescription:

```json
{
  "power_accuracy": null,
  "duration_complete": null,
  "interval_consistency": "<computed>",
  "hr_appropriateness": "<computed>"
}
```

Score = `(interval_consistency x 0.50 + hr_appropriateness x 0.50) x 10`

If no work intervals exist (pure Z2 ride):
- Score = 7.0 if duration > 30min, 6.0 if 20-30min, 5.0 if < 20min
- Note intensity control: was the ride actually Z2? (NP < 75% FTP = good Z2 execution)

Note: "No prescription on file for this session, using objective scoring"

**IMPORTANT**: `factors` must ALWAYS be an object — never `null`. Only exception: Case 2 (races/group rides) where both `score: null` and `factors: null`.

## FTP Test Detection

Before compliance scoring, check if this ride may be an FTP test:

**Detection triggers:**
1. Activity name contains "FTP", "Ramp Test", "20min Test", "MAP Test", "Max Aerobic"
2. OR: Prescription has `type: "test"`
3. OR: A single work interval >= 20min with average watts > current_ftp x 0.95

**If detected:**
Calculate estimated new FTP:
- **20min test**: 20min_avg_watts x 0.95
- **Ramp test**: last 1min avg watts x 0.75
- **8min test**: 8min_avg_watts x 0.90

Display:
```
FTP Test Result
===============
20min avg power: {watts}W
Estimated new FTP: {new_ftp}W ({delta:+d}W from {old_ftp}W)
New W/kg: {new_ftp/weight:.2f} (target: {target_wpkg})
Progress: {progress_pct}% toward target {target_ftp}W

Update FTP? Confirm and all zones will be recalculated.
```

If confirmed: Update `current_ftp` in `data/athlete.json` and recalculate all zone boundaries.
Skip normal compliance scoring — FTP tests are self-scored.

## Post-review actions (IMPORTANT)

### 1. Write compliance data
Read existing `data/logs/compliance.json`, add the new entry keyed by activity_id (as string), and write back:

```json
{
  "17537169660": {
    "date": "2026-02-26",
    "activity_name": "SST 2x20min",
    "prescribed_workout": "SST 2x20min",
    "score": 7.5,
    "factors": {
      "power_accuracy": 0.85,
      "duration_complete": 1.0,
      "interval_consistency": 0.88,
      "hr_appropriateness": 0.90
    },
    "note": "Power slightly below target on interval 2 — good overall execution"
  }
}
```

## Coaching Notes

Structure the coaching notes with clear sections:

**What went well:**
- Specific things that went well (with data to back it up)

**Areas to improve:**
- Specific, actionable improvements for next time

**Significance for your goal:**
- How this session contributes to the long-term goal (e.g., "Z4 performance ({watts}W = {wpkg} W/kg) shows FTP is progressing, {delta}W remaining to Phase target of {ftp_range[1]}W")

**Next steps:**
- Recovery recommendation for next 24-48 hours
- When next hard session should be
- Whether to modify upcoming prescription based on today's performance
