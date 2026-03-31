# /reprostuff

A Claude Code skill that checks your GitHub for new or updated Claude Code skills, compares them against what's installed locally, and offers to bring the good stuff home.

## What It Does

Run `/reprostuff` and it will:

1. Scan a hardcoded GitHub account for repos that look like Claude Code skills
2. Compare against your installed skills
3. Show you what's new, what's changed, and what's current
4. Offer to install or update anything interesting — with reasons, not just yes/no prompts
5. Remember what it saw, so next time it only tells you what changed

## Output Format

The report has three sections:

- **Skill Updates** — new and changed skills with recent commit summaries. The headline.
- **What Else Is Going On** — brief narration of non-skill repo activity. The color.
- **Recommendations** — specific reasons to install or update, if anything's worth grabbing.

If nothing changed since last run, it says so and moves on. Scales the sass to the time interval.

## State Memory

The skill saves a `last_run.json` in its own directory after each run. Next time it runs — even after conversation compaction — it diffs against this snapshot and only reports changes. No full inventory dump every time.

## Installation

```bash
mkdir -p ~/.claude/skills/reprostuff
cp SKILL.md ~/.claude/skills/reprostuff/
```

Then edit the GitHub account in `SKILL.md` to point at your own. It ships pointed at `thereprocase` because that's where it was born, but the whole point is that it's yours to aim wherever you want.

## Configuration

Open `SKILL.md` and change `thereprocase` to your GitHub username. That's it. Everything else adapts.

## License

MIT
