# Velo Coach Skills

## Overview
AI-powered cycling coaching system for Claude Code. Claude acts as a professional cycling coach, communicating via markdown in conversation. Athlete config and training plan are stored in `data/` — read those files for current FTP, zones, and phase info.

## Coaching Workflow
The athlete communicates in natural language. Claude should interpret intent and invoke the appropriate skill automatically:

| What the athlete might say | Skill | Description |
|---------------------------|-------|-------------|
| "sync my data" "update data" | `/sync` | Pull latest intervals.icu data |
| "how am I doing" "can I train today" | `/status` | PMC/TSB quick check + RPE recording + goal progress |
| "how was yesterday's ride" "review that session" | `/ride-review` | Detailed interval analysis + compliance scoring + cardiac drift |
| "what's next" "prescribe a workout" | `/plan` | Prescribe workouts based on current fitness and training phase |
| "I'm tired" "slept badly" "legs are sore" | (embedded in `/status`) | Record RPE to rpe.json + adjustment advice |
| "recovery advice" "what should I eat" "nutrition" | `/recovery` | Personalized recovery protocol + precise nutrition targets |
| "how was this week" "weekly summary" | `/weekly-review` | Weekly training summary: TSS vs target, zone distribution, compliance trend |
| "how was that race" "race prep" | `/race` | Race analysis (surge/pacing/tactical) or pre-race preparation |

**IMPORTANT**: Always invoke the matching skill via the Skill tool when the athlete's message matches one of the above intents. The skill files contain detailed prompts for how to read data and format the coaching response.

### Auto-Sync Check
Before running `/status`, `/ride-review`, `/plan`, `/recovery`, `/weekly-review`, or `/race`, check `data/synced/.last_sync`:
- If last sync was **more than 6 hours ago** -> run `/sync` first automatically
- This ensures coaching advice is always based on fresh data

## Data Architecture

### Athlete Config (single source of truth)
- `data/athlete.json` — FTP, weight, target, zones, max HR (all skills read from here)
- `data/training-plan.json` — Periodization plan with phases, weekly TSS targets, key workouts. May not exist for all athletes

### Synced Data (from intervals.icu API)
- `data/synced/wellness.json` — Daily CTL/ATL/TSB, TSS, eFTP
- `data/synced/power-curves.json` — Best power at key durations: 5s-60min (42d + 1y)
- `data/synced/activities.json` — Activity summaries with power/HR/duration
- `data/synced/laps/{activity_id}.json` — Per-interval breakdown with zone/type/training_load
- `data/synced/.last_sync` — Timestamp of last sync
- `data/synced/.backup/` — Auto-backup of previous sync data (last 5 versions)

### Coaching Logs (locally generated)
- `data/logs/rpe.json` — Daily subjective RPE: fatigue, legs, motivation, sleep (1-10 scale, recorded via conversation)
- `data/logs/prescriptions.json` — Workout prescriptions written by `/plan` (keyed by date)
- `data/logs/compliance.json` — Compliance scores written by `/ride-review` (keyed by activity_id)
- `data/logs/coaching-notes.json` — Mid-week plan adjustments with reasons (illness, injury, schedule change)

### Coaching Data Flow
```
/plan -> prescriptions.json -> (athlete rides) -> /ride-review -> compliance.json
                                                      |                              |
user reports feeling -> /status -> rpe.json -> influences /plan decisions    /recovery (post-ride nutrition)
                                                      |
/sync -> power-curves.json -> displayed in /status
```

### Sync Script
- `bash scripts/sync.sh` — Syncs data from intervals.icu
- Fetches wellness (90 days), power curves, activities (90 days), and intervals/laps
- First sync fetches 90 days of laps; subsequent syncs fetch last 14 days (cached)
- Validates JSON before overwriting, backs up previous data

## Data Validation (all coaching skills)
Before analyzing data, verify:
- `athlete.json` exists and has `current_ftp` > 0 — if not, abort with "WARNING: athlete.json missing required data"
- `wellness.json` / `activities.json` have entries — if empty, suggest running /sync first
- Null field fallbacks: `average_watts` null -> use `weighted_average_watts`; `kilojoules` null -> estimate avg_watts x duration_s / 1000
- **TSS source**: Use `ctlLoad` from wellness.json for daily TSS (computed by intervals.icu from power data). Do NOT use Strava's `suffer_score` (that's "Relative Effort", not TSS). For per-activity TSS: `TSS = IF^2 x hours x 100` where `IF = NP / FTP`

## Monthly Training Log
When the user asks for a monthly summary or at month end, generate `data/logs/YYYY-MM.md`:
1. Header: athlete name, weight, FTP, plan phase
2. Monthly summary: total rides, total TSS, avg daily TSS, CTL change, hours
3. Weekly summaries: TSS vs target, session count, rating
4. Power curve snapshot: key durations, 42d vs 1y
5. Daily entries: reverse chronological, ride data per day
6. Key achievements: PRs, milestones

## Key Conventions
- **CRITICAL: Always run `date '+%Y-%m-%d %A'` to get the real date/day-of-week. NEVER trust the system-provided `currentDate` context — it can be wrong.**
- Power zones: Coggan 7-zone model — read from `data/athlete.json`, never hardcode
- TSS/CTL/ATL/TSB follow standard PMC formulas (CTL 42-day, ATL 7-day exponential decay)
- When FTP changes, update `data/athlete.json` — all skills will pick up the new value automatically
- RPE scale: 1-10 (10 = best/freshest), recorded conversationally, stored in `data/logs/rpe.json`
- Compliance scoring: prescribed vs actual (power accuracy 40%, duration 25%, consistency 20%, HR 15%)
