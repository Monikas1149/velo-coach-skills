Run the data sync script to download latest training data from intervals.icu into local JSON files.

## Usage
- `bash scripts/sync.sh` — Full sync (wellness, power curves, activities, intervals)

## Pre-sync Checks

Before running sync, verify:
1. **Check `.env` exists** — if not, warn: "WARNING: .env not found — sync requires API keys"
2. **Check data freshness** — Read `data/synced/.last_sync` and check the timestamp
   - If data is from today: "Data is already current. Sync anyway?" (proceed if user confirms or if called as auto-sync from another skill)
3. **Run `date '+%Y-%m-%d %H:%M %A'`** to get current time for reporting

## Running the Sync

Execute: `bash scripts/sync.sh`

The script:
- Downloads wellness/PMC data from intervals.icu (90 days)
- Downloads power curves from intervals.icu
- Downloads activities from intervals.icu (90 days)
- Fetches interval/lap details per activity
- Writes all data to local JSON files in `data/synced/`

## Post-sync Validation

After sync completes, verify the data quality:
1. `Read data/synced/wellness.json` — check last entry has today's or yesterday's date
2. `Read data/synced/activities.json` — check most recent activity date
3. `Read data/synced/power-curves.json` — confirm power curves updated

If any data is missing expected recent entries, warn the user.

## Sync Report

After sync, provide a brief report:

```
Sync complete
{current_datetime}
|- Wellness: latest {latest_date}
|- Activities: latest {latest_activity_name}, {latest_date}
|- Laps: updated
|- Power curves: updated
```

If errors occurred during sync, report them clearly:
```
WARNING: Sync partially failed:
- intervals.icu wellness: OK
- intervals.icu activities: OK
- Intervals/laps: WARNING - 3 activities failed
Suggest retrying: bash scripts/sync.sh
```

## Silent Mode (Auto-sync)

When sync is triggered automatically by other skills (status, ride-review, plan, recovery):
- Skip the confirmation prompt
- Use a shorter report format: "Auto-synced ({N} new activities)"
- If sync fails, still proceed with existing data but note: "WARNING: Sync failed, using existing data"
