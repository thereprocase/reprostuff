# /reprostuff

A Claude Code skill built for [Repro](https://github.com/thereprocase). It checks Repro's GitHub for new or updated Claude Code skills, compares them against what's installed locally, and offers to bring the good stuff home.

## What It Does

Run `/reprostuff` and it will:

1. Scan Repro's GitHub account for repos that look like Claude Code skills
2. Compare against locally installed skills
3. Show what's new, what's changed, and what's current
4. Offer to install or update anything interesting — with reasons, not just yes/no prompts
5. Remember what it saw, so next time it only reports what changed

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

This skill is hardcoded to Repro's GitHub account (`thereprocase`). It was built for one person and it knows who that person is.

## License

MIT
