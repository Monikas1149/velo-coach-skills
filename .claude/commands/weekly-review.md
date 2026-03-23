Weekly training review — the most important check-in for long-term progression. This is where patterns become visible: are we on track for the target? Is training distribution correct? Is recovery adequate?

## When to trigger

Use this skill when the user asks for:
- "how was this week" / "weekly summary" / "weekly review"
- Any request about the current or past week's training overview
- Automatically suggest on Sundays or when week ends

## Data to read

1. Run `date '+%Y-%m-%d %A'` to get today's date and day of week
2. `Read data/athlete.json` — FTP, weight, target, max_hr
3. `Read data/training-plan.json` — Current phase, weekly TSS targets
4. `Read data/synced/wellness.json` — Daily CTL/ATL/TSB for the week
5. `Read data/synced/activities.json` — All activities from the current week (Monday to today)
6. `Read data/synced/laps/{activity_id}.json` — Lap details for NP/IF of key sessions
7. `Read data/logs/compliance.json` — This week's compliance scores
8. `Read data/logs/prescriptions.json` — This week's prescriptions (check for `revision` fields — revised prescriptions are the source of truth)
9. `Read data/synced/power-curves.json` — For power profile tracking
10. `Read data/logs/coaching-notes.json` — Mid-week adjustments (illness, injury, schedule changes)

## Determine the week boundaries

- Calculate Monday of the current week: `today - (day_of_week - 1) days`
- Week = Monday 00:00 to Sunday 23:59
- If the user asks about "last week", shift back 7 days
- Current training plan week: `floor((today - plan_start) / 7) + 1`

## TSS Data Source

**IMPORTANT**: For daily TSS totals, use `ctlLoad` from wellness data — this is the actual TSS calculated by intervals.icu from power data. Do NOT use Strava's `suffer_score` from activities (that's "Relative Effort", not TSS). Sum the `ctlLoad` values for each day of the week.

For per-activity TSS estimates, compute from NP/FTP: `TSS = IF^2 x duration_hours x 100` where `IF = NP / FTP`.

## Week Completeness

- Count how many days have elapsed: `days_elapsed = min(7, today_day_of_week)` (Mon=1, Sun=7)
- If days_elapsed < 7, note: "As of {today} ({days_elapsed}/7 days)"
- For TSS target comparison with incomplete week: pro-rate the target
  - `prorated_target_min = target_min x days_elapsed / 7`
  - `prorated_target_max = target_max x days_elapsed / 7`
- Show both actual vs prorated and projected full-week estimate

## Report Format

### Week {week} Review — Phase {id}: {name}
**{monday_date} to {sunday_date}** {if incomplete: "(as of {today}, {days_elapsed}/7 days)"}

---

### Weekly Volume

| Metric | Actual | Target (prorated) | Achievement |
|--------|--------|-------------------|-------------|
| TSS | {actual_tss} | {prorated_target_min}-{prorated_target_max} (full week: {full_target}) | {pct}% |
| Ride Time | {hours}hr | {target_hours_min}-{target_hours_max}hr | {pct}% |
| Sessions | {count} | — | |
| Rest Days | {rest_days} | >=1-2 | {ok/warning} |
| kJ Spent | {total_kj} | — | |

**TSS vs Target assessment:**
- < 80% of target range minimum -> "WARNING: Training volume insufficient — may impact Phase {id} progress"
- 80-100% of minimum -> "OK: Training volume adequate"
- Within target range -> "EXCELLENT: Right on target"
- > 110% of maximum -> "WARNING: Over target — recommend reducing load next week"

### PMC Trend

Show CTL/ATL/TSB progression through the week:

```
       Mon   Tue   Wed   Thu   Fri   Sat   Sun
CTL    XX.X  XX.X  XX.X  XX.X  XX.X  XX.X  XX.X
ATL    XX.X  XX.X  XX.X  XX.X  XX.X  XX.X  XX.X
TSB    XX.X  XX.X  XX.X  XX.X  XX.X  XX.X  XX.X
```

**Key metrics:**
- CTL change this week: +{delta} (vs last week's change: +{prev_delta})
- Ramp rate: {ramp_rate} TSS/day
- CTL trajectory: At this rate, estimated CTL at Phase end = {projected_ctl}

### Training Distribution

Estimate time in each zone from activity NP and duration. For each ride:
- If NP < Z2 max -> classify as Z1-Z2
- If NP in Z3 -> classify as Z3
- If NP in Z4+ -> classify as Z4+

Show distribution:
```
Zone Distribution (estimated):
Z1-Z2 (Aerobic):     [================    ]  4.2hr (64%)
Z3 (Tempo):          [====                ]  1.1hr (17%)
Z4+ (High Intensity):[===                 ]  0.8hr (12%)
Other (Warm-up/Cool):[=                   ]  0.5hr (7%)
```

**Phase-appropriate distribution check:**
| Phase | Ideal Z1-Z2 | Ideal Z3 | Ideal Z4+ |
|-------|-------------|----------|-----------|
| 1 (Base) | 75-80% | 15-20% | 5-10% |
| 2 (SST) | 65-70% | 20-25% | 10-15% |
| 3 (Threshold) | 60-65% | 15-20% | 20-25% |
| 4 (Specialization) | 55-60% | 10-15% | 25-35% |
| 5 (Peak) | 70-75% | 10-15% | 15-20% |

Compare actual vs ideal: "Z4+ time {actual}% vs ideal {ideal}% — {assessment}"

### Daily Breakdown

For each day of the week, show:

```
| Day | Activity | TSS | NP (W/kg) | IF | Compliance | Notes |
|-----|----------|-----|-----------|-----|------------|-------|
| Mon | Rest | 0 | — | — | — | Rest day |
| Tue | SST 2x15min | 72 | XXXW (X.XX) | 0.91 | 8.2 | Good |
| Wed | Z2 Endurance | 45 | XXXW (X.XX) | 0.65 | 7.5 | |
| Thu | VO2max 3x3min | 65 | XXXW (X.XX) | 1.10 | — | No prescription |
| Fri | Z2 Easy | 35 | XXXW (X.XX) | 0.59 | — | |
| Sat | Long Ride 2hr | 85 | XXXW (X.XX) | 0.72 | 7.8 | |
| Sun | Rest | 0 | — | — | — | Recovery |
| | **Total** | **302** | | | **avg 7.8** | |
```

### Training Stress Metrics (Training Monotony & Strain)

Calculate from 7 daily TSS values (use ctlLoad from wellness):
- **Daily TSS mean**: `avg_tss = sum(daily_tss) / 7`
- **Daily TSS stdev**: standard deviation of 7 daily TSS values (rest days = 0)
- **Monotony**: `avg_tss / stdev_tss` — measures training variety
  - Monotony > 2.0: "WARNING: Training monotony too high — daily loads too similar, increases overtraining risk. Add more variety in session intensity"
  - Monotony 1.5-2.0: "OK: Moderate training variation"
  - Monotony < 1.5: "Good: Sufficient training variety"
- **Strain**: `weekly_tss x monotony` — combined load + monotony metric
  - Strain > 3000: "WARNING: Training strain too high — recommend reducing load 15-20% or increasing variety next week"

High monotony is the hidden danger — it means every day feels roughly the same, which accumulates systemic fatigue without adequate recovery stimulus. A good week has variety: a hard day, an easy day, a rest day, an interval day.

Display only if anomalous (monotony > 2.0 or strain > 3000). Otherwise skip.

### Key Data

**Best performances** (highlight the week's best efforts):
- Best NP: {watts}W ({wpkg} W/kg) in {activity_name}
- Best interval: {watts}W for {duration} in {activity_name}
- Highest TSS single session: {tss} in {activity_name}

**Power progress tracking** (compare to power-curves data):
```
Duration   This Week   42d Best   Target {target_wpkg} W/kg
5s         {watts}W    {42d}W     {target needed}W
1min       {watts}W    {42d}W     {target needed}W
5min       {watts}W    {42d}W     {target needed}W
20min      {watts}W    {42d}W     {target needed}W
```

### Compliance Trend

Average compliance this week: {avg_score}/10
Compliance trend (last 4 weeks): {week-4} -> {week-3} -> {week-2} -> {week-1} -> **{this_week}**

If compliance is declining:
- "WARNING: Compliance declining — targets may be too high, consider adjusting prescription intensity"

If compliance is improving:
- "OK: Compliance improving — training targets are well calibrated"

### Goal Progress

```
{target_wpkg} W/kg Target
==========================
FTP: {current}W -> {target}W (gap: {gap}W)
W/kg: {current_wpkg} -> {target_wpkg}

Phase {id} target: {phase_ftp_min} -> {phase_ftp_max}W
Phase progress: Week {week_in_phase} / {phase_weeks} weeks total

This week CTL change: +{ctl_delta}
At this rate, estimated CTL at Phase end: {projected_ctl}
```

### Coach's Summary

**This week's highlights:**
- {1-2 specific positive observations with data}

**Areas to improve:**
- {1-2 specific areas to improve}

**Next week plan:**
- Week {next_week} plan adjustments (if any)
- Deload reminder (if next week is week 4/8/12/etc.)
- Key sessions to focus on
- Recovery priority items

**Key takeaway:**
> {Motivational but data-backed summary, connecting this week to the long-term goal}
