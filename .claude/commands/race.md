Race analysis and pre-race preparation. This skill provides race-specific coaching that the standard ride-review doesn't cover: pacing strategy, surge analysis, tactical decisions, and race-day preparation.

## When to trigger

Use this skill when:
- User asks to review a race ("how was that race", "race review")
- Activity name contains "Race", "Crit", "Time Trial", "Gran Fondo"
- User asks for race preparation ("race tomorrow", "race day prep", "race strategy")
- User asks about race tactics or pacing

## Data to read

1. `Read data/athlete.json` — FTP, weight, max_hr, target_ftp
2. `Read data/synced/activities.json` — Find the race activity
3. `Read data/synced/laps/{activity_id}.json` — Detailed lap/interval data
4. `Read data/synced/wellness.json` — TSB on race day (freshness context)
5. `Read data/synced/power-curves.json` — For benchmark comparisons
6. `Read data/logs/prescriptions.json` — Check if race was planned
7. `Read data/training-plan.json` — Current phase context

---

## Part A: Post-Race Analysis

### Race Report: {race_name}
**{date}**

### Race Overview

| Metric | Value | Interpretation |
|--------|-------|---------------|
| Duration | {duration} | |
| NP | {np}W ({np/weight:.1f} W/kg) | {comparison to FTP} |
| Avg Power | {avg}W ({avg/weight:.1f} W/kg) | |
| Max Power | {max}W ({max/weight:.1f} W/kg) | Sprint/attack |
| IF | {np/ftp:.2f} | {interpretation} |
| VI | {np/avg:.2f} | {variable vs steady interpretation} |
| TSS | {tss} | |
| Avg HR | {hr} ({hr/max_hr*100:.0f}% MHR) | |
| Max HR | {max_hr_actual} ({max_hr_actual/max_hr*100:.0f}% MHR) | |
| Avg Cadence | {rpm} | |

**Race intensity context:**
- IF comparison: IF > 0.95 for < 60min = strong race effort
- If NP > FTP: race was at or above threshold — good fitness indicator
- W/kg context: "Race NP {np_wpkg} W/kg vs target {target_wpkg} W/kg — {gap analysis}"

### Race Readiness (display prominently before detailed analysis)

- TSB on race day: {tsb} — {interpretation}
  - TSB > 0: "FRESH — good condition for all-out racing"
  - TSB -5~0: "SLIGHTLY FATIGUED but raceable"
  - TSB -10~-5: "FATIGUED — performance may be limited 3-5%"
  - TSB < -10: "SIGNIFICANTLY FATIGUED — race performance does not represent true ability, recommend 48hr taper before next race"
- Note: "At this TSB, estimated ~{abs(tsb)/3:.0f}% performance limitation"

**Warm-up Analysis:**
If same-day activities exist before the race, analyze them:
- Total warm-up time and average power
- Appropriate length? (20-30min ideal, 30-60min acceptable, >60min too long)
- Were openers included? (short Z5-Z6 bursts in warm-up = good activation)

### Surge & Attack Analysis

**Definition**: A "surge" = power exceeds 120% FTP (>{ftp*1.2}W) for >=10 seconds

**If only 1 lap available** (common for virtual races — the entire race as a single lap):
- Cannot enumerate individual surges from lap data
- Estimate surge characteristics from aggregate metrics:
  - VI > 1.10 -> "Race had significant power variability, likely multiple surges"
  - Max power vs NP ratio: higher = sharper surges
  - NP - Avg Power delta: larger = more variability
- Compare max power to power curve bests: "Max {max}W = {max/5s_best*100:.0f}% of 5s best — {deployed well / under-deployed}"
- Skip the detailed surge distribution table; show the estimates instead

**If multiple laps/intervals available**, scan and count surges:

```
Surge Analysis:
Total surges: {count}
Average surge power: {avg_surge_watts}W ({wpkg} W/kg)
Average surge duration: {avg_duration}s
Max surge: {max_surge_watts}W for {max_surge_duration}s

Surge Distribution:
 #1:  XXXW x 15s  (race start positioning)
 #2:  XXXW x 25s  (climb attack)
 #3:  XXXW x 12s  (counter-attack)
 #4:  XXXW x 8s   (sprint finish)
```

**Surge consistency**: stdev of surge power / mean surge power
- < 15% = very consistent surges (good)
- 15-25% = moderate (normal for races)
- > 25% = inconsistent (might be pacing poorly or fading)

### Pacing Analysis

Split the race into halves (by time) and compare:

```
Pacing Split:
          1st Half    2nd Half    Delta
NP:       {np1}W      {np2}W     {pct_change}%
Avg HR:   {hr1}       {hr2}      {hr_change}
Cadence:  {rpm1}      {rpm2}     —
```

**Pacing strategy assessment:**
- Negative split (2nd half NP higher): "EXCELLENT pacing — stronger in second half, had reserves for the finish"
- Even split (within +/-3%): "GOOD — steady pacing, well controlled"
- Positive split (1st half NP > 5% higher): "WARNING: Started too fast, power faded — recommend more conservative start next time"
- Fade > 10%: "SIGNIFICANT fade — starting intensity was too high or insufficient fueling"

**HR-Power decoupling** (aerobic efficiency under race stress):
```
EF_first_half = NP_first / avg_HR_first
EF_second_half = NP_second / avg_HR_second
decoupling = (EF_first - EF_second) / EF_first x 100
```
- < 5%: Excellent aerobic capacity for race intensity
- 5-10%: Normal race fatigue
- > 10%: Significant fatigue — consider improving base fitness or pre-race fueling

### Tactical Assessment

**Drafting efficiency** (flat/rolling courses):
- Low VI (< 1.10) on flat sections = good draft usage
- High VI on flats = not sitting in, wasting energy
- "VI {vi} on flat sections — {draft_assessment}"

**Climb performance**:
- For any interval on climbs (identified by high power sustained >2min):
- W/kg on climb: "Climb: {watts}W = {wpkg} W/kg for {duration}"

**Sprint performance** (last 30s of race or max power effort):
- Peak 5s power: {watts}W ({wpkg} W/kg)
- Peak 15s power: {watts}W ({wpkg} W/kg)
- Sprint timing: "Sprint in final {distance}m — {too early/perfect/late}"

### Race vs Training Comparison

```
                Race Day    Recent Training    Gap
5s power:       {race}W     {training}W       {delta}
1min power:     {race}W     {training}W       {delta}
5min power:     {race}W     {training}W       {delta}
20min power:    {race}W     {training}W       {delta}
NP (race avg):  {race}W     —                 —
```

Did the race bring out better performance than training? This is normal — adrenaline + competition typically adds 3-8%. Note: "Race performance is typically 3-8% higher than training — this is the normal competitive effect"

### Race Coaching Notes

**Tactical highlights:**
- {specific good decisions with power data}

**Areas to improve:**
- {specific tactical improvements}

**Performance bottleneck:**
- Which power duration limited race performance?
- "Your 5min power ({wpkg} W/kg) is the bottleneck — VO2max training needed to improve climbing"
- Or: "Sprint power ({5s_wpkg} W/kg) is sufficient, but FTP ({ftp_wpkg} W/kg) limits your ability to stay with the main group"

**Next race recommendations:**
- Specific tactical adjustments
- Training focus areas to improve race performance
- Pacing strategy modification

---

## Part B: Pre-Race Preparation

When the user mentions an upcoming race ("race tomorrow", "race prep"):

### Taper Science

A proper taper reduces fatigue (ATL drops) while maintaining fitness (CTL stays). Research shows the ideal taper:
- **Duration**: 7-14 days for A-priority races, 3-5 days for B-priority, 1-2 days for weekly races
- **Volume reduction**: Reduce volume by 40-60%, but MAINTAIN intensity. This is counterintuitive but critical — the key sessions during taper should include short, sharp efforts (Z5-Z6) to keep neuromuscular activation high
- **Frequency**: Maintain training days (don't just stop riding), but shorten each session
- **TSB target**: Aim for TSB +5 to +15 on race day. Below 0 = still fatigued. Above +20 = may have lost sharpness (detraining)

**Taper prescription template** (adjust based on days until race):
- D-7 to D-5: Normal training volume at 60%, include 1 hard session with race-pace intervals
- D-4: Easy Z2 ride, 45min
- D-3: Openers session (short Z5-Z6 efforts within Z2 ride)
- D-2: Complete rest or very easy 20min spin
- D-1: Openers (see below)
- Race day: Race!

**Common taper mistakes:**
- "I've been resting for days, why do my legs feel heavy?" -> Normal! Day 2-3 of taper often feels WORSE — muscles are repairing. Trust the process, race-day legs will be there
- Training through nervousness: Athletes often add extra rides because they're anxious. Resist this — the fitness gains from a ride 2 days before race are zero, but the fatigue cost is real
- Carb loading too late: Start 48hr before, not the night before

### Race Day Prep: {race_name}

**D-1 (day before):**
- Openers: 30-40min Z2 + 3-4x30s Z6 (130-140% FTP) + 2x15s Z7 (>150% FTP)
  - Purpose: Activate neuromuscular system without accumulating fatigue
  - TSS target: < 35
- Carb loading: 7-8 g/kg = {weight*7.5:.0f}g carbs
  - Breakfast: High-carb (oatmeal/rice/cereal)
  - Lunch: Carbs + protein (pasta + chicken)
  - Dinner: Carb-focused, low fiber (white rice + fish/eggs)
  - Snacks: bananas, energy bars
- Hydration: 3L+ (urine should be light yellow)
- Sleep: 8hr+ (race day sleep matters less than D-2 sleep)
- Avoid: alcohol, high-fiber food, new foods, high-intensity training

**Race Day:**
- Pre-race meal (2-3hr before):
  - Carbs 1-2g/kg = {weight*1.5:.0f}g (white rice/toast/banana)
  - Avoid high-fiber, high-fat
- Caffeine (60min before): 3-5 mg/kg = {weight*4:.0f}mg (approx {weight*4/80:.0f} cups coffee)
  - Only if you normally drink coffee — don't try new things on race day
- Warm-up (20-30min before race start):
  ```
  10min Z2 (65-72% FTP)
  3x1min progressive: Z3 -> Z4 -> Z5
  2x15s Z6 sprint (130% FTP)
  5min easy Z1
  ```
- 500mL water + electrolytes during warm-up
- Race gel: 1 gel 15min before start (if race > 30min)

**Race Strategy** (based on current fitness and power profile):

```
Your race profile:
FTP: {ftp}W ({ftp/weight:.1f} W/kg)
5min: {5min_power}W ({5min/weight:.1f} W/kg)
1min: {1min_power}W ({1min/weight:.1f} W/kg)
5s:   {5s_power}W ({5s/weight:.1f} W/kg)

Strategy based on strengths:
{if 5s power is strong relative to FTP}
-> Sprinter type: sit in the group, all-out sprint in final 200m
{elif 5min is strong relative to FTP}
-> Climber type: push hard on climbs to create gaps
{elif FTP is the strongest}
-> Tempo rider: steady high output, wear down opponents through sustained effort
{end}
```

**Pacing target:**
- Target NP: {ftp * race_if}W (IF {race_if})
  - For < 30min race: IF 0.95-1.05
  - For 30-60min race: IF 0.85-0.95
  - For 60-90min race: IF 0.80-0.90
  - For > 90min race: IF 0.75-0.85
- Surge budget: save {N} hard efforts for key moments (climbs, attacks, sprint)
- Don't chase every attack — sit in, conserve, and respond only to dangerous moves

### Race Physiology Insights

**Anaerobic Work Capacity (W')**:
W' represents the total work you can do above FTP before exhaustion. In races, every surge above FTP "spends" W', and it recharges (slowly) when you ride below FTP.
- Estimate W' expenditure: `sum of (watts_above_FTP x duration) for each surge`
- If total W' spent is very high and rider still finished strong -> W' is likely a strength (good anaerobic capacity)
- If rider faded after heavy W' spending -> needs to manage surges better or improve W' through VO2max training
- Recovery rate matters: short inter-surge rest periods mean less W' recovery. Riders with higher FTP relative to race pace recover W' faster (more headroom below FTP)

**Race-Specific Fueling Assessment**:
- For races > 45min: did the athlete fuel during? (Check if pre-race notes mention this)
- Bonk indicators: power fade in final 20% combined with HR drop = likely glycogen depletion
- "If power faded + HR dropped in the finale = likely insufficient fueling. Next race: take 30-40g carbs every 20min"

### Race Day Contingency Strategies

**If legs feel dead during warm-up:**
- Don't panic — extend warm-up by 5min with progressive efforts
- Add 2 extra Z5 openers (10s each) to wake up the legs
- Start race conservatively, wait for legs to come around (usually 10-15min in)

**If dropped from main group:**
- Don't chase alone — the energy cost of bridging solo is enormous
- Find the next group and work with them
- Treat it as a training effort: steady tempo, practice pacing

**If race is much harder than expected:**
- Switch to survival mode: sit at the back, minimal surging
- Only respond to climbs and finish line — save everything for those moments
- Fuel aggressively if >30min remains (gel every 20min)

### Post-race actions

After the race:
1. Save compliance entry with `score: null, factors: null` (races aren't scored for compliance)
2. Suggest `/recovery` for post-race nutrition and recovery protocol
3. Note: "Racing is excellent high-intensity training — TSS {tss} equivalent to a hard interval session"
