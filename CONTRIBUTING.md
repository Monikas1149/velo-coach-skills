# Contributing to velo-coach-skills

Thanks for your interest in improving the cycling coach! Here's how to contribute.

## How it works

This project is a collection of **Claude Code skill files** (`.claude/commands/*.md`). Each skill is a markdown prompt that teaches Claude how to analyze cycling data and provide coaching advice.

When a user types something like "how was my ride?" in Claude Code, it matches the intent to a skill file and follows the instructions inside to read data, run calculations, and generate coaching output.

## Ways to contribute

### 1. Improve existing skills

The skill files contain training science knowledge. If you spot inaccuracies, outdated recommendations, or missing nuances:

- **Recovery protocols** (`recovery.md`) — nutrition timing, supplement evidence, sleep optimization
- **Workout library** (`plan.md`) — add workouts, improve progression logic
- **Compliance scoring** (`ride-review.md`) — refine the algorithm weights or thresholds
- **Safety alerts** (`status.md`) — add new overtraining detection heuristics

### 2. Add new skills

Ideas for new skills:

- `/weight` — weight trend analysis with race weight targets
- `/gear` — equipment tracking and maintenance reminders
- `/goals` — long-term goal planning with milestone tracking
- `/compare` — compare two time periods (e.g., this month vs last month)

### 3. Add platform integrations

The sync script currently only supports intervals.icu. PRs welcome for:

- **TrainingPeaks** API integration
- **Garmin Connect** data sync
- **Wahoo SYSTM** workout import
- **Strava** API integration (requires OAuth setup)

### 4. Improve documentation

- Better setup guides for different platforms
- Video walkthroughs
- Translations (skill output language, not the skill prompts themselves)

## Development workflow

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/my-improvement`)
3. Make your changes
4. Test with Claude Code: `cd velo-coach-skills && claude`
5. Verify your skill works: try the natural language triggers listed in CLAUDE.md
6. Submit a PR with a clear description of what changed and why

## Skill file format

Each skill file in `.claude/commands/` follows this pattern:

```markdown
[Brief description of what this skill does]

## Data to read
[List of files Claude should read before analyzing]

## Analysis logic
[Step-by-step instructions with formulas]

## Output format
[How to structure the coaching response]
```

Key rules:
- **Never hardcode power values** — always use `{ftp * percentage}` or "X% FTP"
- **Read zones from `data/athlete.json`** — never hardcode zone boundaries
- **Keep formulas exact** — compliance scoring, TSS calculations, etc. must be precise
- **Include null safety** — always handle missing data gracefully

## Code of conduct

Be respectful, constructive, and kind. We're all here to ride faster and recover better.
