Comprehensive coaching status report — the athlete's daily check-in with their coach.

## Beginner Detection

After loading athlete config and activities:
- If `current_ftp` < 150 AND total activities < 25 -> **beginner mode**
- Beginner mode adjustments:
  - Use simpler language (fewer technical terms)
  - Ramp rate safety threshold: > 5 = caution (not > 7)
  - TSB recovery threshold: < -15 = recommend rest (not -20)
  - Focus coaching on **consistency and habit-building**, not performance optimization
  - Skip W/kg goal section and power profile analysis

## Null-Safety: training-plan.json

If training-plan does not exist OR `plan_start` is null:
- Skip the "Training Phase" line in today's status
- Replace with: "No structured training plan — focus on building habits and aerobic base"
- Skip phase references in coaching advice — use general TSB-based advice only
- Skip W/kg milestone timeline

## Data to read

1. `Read data/athlete.json` — Current FTP, weight, target_ftp, target_wpkg, max_hr, peak_ftp
2. `Read data/training-plan.json` — Current phase (calculate week from plan_start)
3. `Read data/synced/wellness.json` — Last 14 entries for PMC trend
4. `Read data/synced/activities.json` — Last 10 activities for recent volume. For accurate NP, cross-reference laps data when available
5. `Read data/synced/laps/{activity_id}.json` — Lap details for NP/IF (when needed)
6. `Read data/synced/power-curves.json` — Power curve data
7. `Read data/logs/compliance.json` — Last 5 entries for compliance trend

## Report format

### Today's Status

- **CTL (Fitness)**: value + trend (up/down/flat) vs 7 days ago + delta
- **ATL (Fatigue)**: value + trend
- **TSB (Form)**: value + interpretation:
  - TSB > 15: "FRESH — may need training stimulus"
  - TSB 5~15: "FRESH — good for high intensity or testing"
  - TSB -10~5: "OPTIMAL — best zone for daily training"
  - TSB -20~-10: "FATIGUED — monitor recovery"
  - TSB < -20: "SIGNIFICANTLY FATIGUED — rest recommended"
- **eFTP**: latest value from wellness sportInfo (if available)
- **Ramp Rate**: CTL_today - CTL_7days_ago. Interpretation:
  - < 3: Conservative (appropriate for recovery periods)
  - 3-5: Moderate (ideal growth rate)
  - 5-7: Aggressive (monitor closely, beginners take care)
  - > 7: WARNING — too fast, recommend 20% reduction next week
- **Training Phase**: Phase name + "Week X / Y weeks total" + phase focus

### Goal Progress Tracker

Show this section for athletes with `target_wpkg` in athlete.json:

```
Goal Progress: {target_wpkg} W/kg
=============================
Current FTP:   {ftp}W -> {ftp/weight:.2f} W/kg
Target FTP:    {target_ftp}W -> {target_wpkg} W/kg
Historical Best: {peak_ftp}W -> {peak_ftp/weight:.2f} W/kg ({peak_ftp_date})

Progress: [========........] XX%
Gap: {delta}W (current {ftp} -> target {target_ftp})

Phase {id} target: {ftp_range[0]} -> {ftp_range[1]}W (Week {week_in_phase}/{phase_weeks})
Phase milestones: {phase targets from training-plan.json}
```

Progress calculation: `(current_ftp - start_ftp) / (target_ftp - start_ftp) x 100`
where start_ftp = FTP at plan_start (use first wellness entry after plan_start, or current_ftp if plan just started)

Milestones: read `ftp_range` from each phase in training-plan.json to show phase targets.

### Power Profile Analysis

If power-curves data exists, show key durations comparing 42-day vs 1-year best:

```
Power Profile
         42d     1y     vs Peak  W/kg
5s       XXX W   XXX W   XX%     XX.X
1min     XXX W   XXX W   XX%      X.X
5min     XXX W   XXX W   XX%      X.X
20min    XXX W   XXX W   XX%      X.X
60min    XXX W   XXX W   XX%      X.X
```

**FTP estimation from power curve** (more reliable than eFTP):
- From 20min: `20min_best x 0.95` (Coggan formula)
- If 20min_42d > current_ftp x 1.02: "20min best suggests FTP may be higher than current setting — consider an FTP test"
- If 20min_42d < current_ftp x 0.90: "42-day 20min best is low — may indicate lack of sustained efforts recently, not necessarily FTP decline. Consider scheduling a 20min test to confirm"
- If eFTP and current_ftp differ by > 5%: "eFTP ({eftp}W) vs set FTP ({ftp}W) differ by {pct}% — consider recalibrating"

**Strength/weakness profile** (compare to target W/kg profile):
- 5s > 10 W/kg -> "Sprint power OK"
- 1min > 6.5 W/kg -> "Anaerobic capacity OK" (else "Needs improvement")
- 5min > 5.5 W/kg -> "VO2max OK" (else "Needs improvement")
- 20min > 4.7 W/kg -> "FTP approaching target" (else "Core gap")

If no power-curves data available, skip this section silently.

### Weight & W/kg

Display:
- Current weight (from athlete.json or most recent wellness entry)
- W/kg = FTP / weight
- Dual-track progress toward target W/kg (power path vs weight path)

### Recent Training Summary (last 7 days)

Scan activities for the last 7 days and summarize:
- Total rides / Total TSS / Total hours / Avg daily TSS
- Training distribution: hours in each zone (estimate from NP and average power)
- Highlight the hardest session (highest NP or kJ)
- Compliance trend: average compliance score from last 5 scored entries

### Safety Alerts (display FIRST if ANY triggered)

Check these conditions and display warnings **before** coaching advice (they override normal advice):

**Load Alerts:**
1. **Ramp Rate > 7**: "WARNING: Training load increasing too fast — ramp rate {value}, recommend 20% reduction next week"
2. **TSB < -30**: "CRITICAL: Severe fatigue — TSB {value}, recommend 2-3 days complete rest"
3. **3+ consecutive days with TSS > 100** (check last 3 activities by date): "WARNING: {N} consecutive high-load days — schedule a recovery day"
4. **Week TSS > phase weekly_tss[1] x 1.1** (if training plan exists): "WARNING: Week TSS {actual} exceeds target ceiling {limit}"
5. **Ramp rate > 5 for more than 2 consecutive weeks**: "WARNING: {N} consecutive weeks of high ramp rate — schedule a recovery week"
6. **Last rest day > 5 days ago** (no day with TSS < 30): "WARNING: {N} days without a full rest day"
7. **Week-over-week TSS jump > 10%**: "WARNING: Week TSS increased {pct}% over last week. Generally recommend <= 10% weekly increase to reduce injury risk"

**Overtraining Red Flags** (check wellness trend data):
8. **eFTP declining for 2+ weeks** while training load is maintained/increasing: "WARNING: eFTP declining for consecutive weeks ({eftp_2wk_ago}->{eftp_now}), possible accumulated fatigue or overtraining precursor"
9. **Power curve 42d vs 1y declining > 10% at key durations** (5min, 20min): "WARNING: 42-day best power down {pct}% from annual best — may need a recovery week"
10. **RPE reporting**: If athlete reports fatigue <= 5 or legs <= 5 for 2+ consecutive check-ins -> "WARNING: Consecutive low RPE scores, body may need a deload"

**Immune & Health:**
11. **TSS > 150 single session + athlete on consecutive hard days**: "NOTE: Post-high-load immune window — stay warm, get adequate sleep, avoid crowds"

For beginners, thresholds are more conservative: ramp > 5, TSB < -15, 2+ consecutive days TSS > 60.

### Training Stress Metrics (Training Monotony & Strain)

If 7 days of TSS data available, calculate:
- **Daily TSS mean** (last 7 days): `avg_tss = sum(daily_tss) / 7`
- **Daily TSS stdev**: standard deviation of last 7 daily TSS values
- **Monotony**: `avg_tss / stdev_tss` — measures training variety
  - Monotony > 2.0: "WARNING: Training monotony too high — daily loads too similar, lacking variation. High monotony + high load = overtraining risk"
  - Monotony 1.5-2.0: OK
  - Monotony < 1.5: Good variety
- **Strain**: `weekly_tss x monotony` — combined load + monotony metric
  - Strain > 3000: "WARNING: Training strain too high — recommend reducing load or increasing variety next week"

Display only if anomalous (monotony > 2.0 or strain > 3000). Otherwise skip to keep status concise.

### Coach's Advice

Based on TSB, current phase, and recent load:

**TSB-based advice**:
- If TSB < -20: "Body needs recovery — recommend Z1 recovery ride or complete rest"
- If TSB -10 to 5: "Good to train" + suggest intensity appropriate for current phase + day of week
- If TSB > 10: "Well rested — good time for high-intensity work"
- Reference current phase goals from training-plan data
- Consider day of week vs weekly_structure from training-plan
- Consider weekly TSS progression target (recommend <= 10% increase week-over-week)

**Always end with a specific actionable suggestion**: "Recommendation for today: ..." with a concrete workout or rest prescription.

Keep it concise — this is a quick check-in, not a full analysis. If the athlete needs a detailed workout plan, suggest running `/plan`.
