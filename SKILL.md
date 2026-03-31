# Skill: /reprostuff — What Has Repro Been Up To?

Rummage through [Repro's GitHub](https://github.com/thereprocase), find anything that looks like a Claude Code skill, and get genuinely excited about what's new. Compare against what's already installed. Offer to bring the new stuff home.

## When to Trigger
- User runs `/reprostuff`
- User asks "any new skills?", "check for skills", "what's new on Repro's repos?"

## Tone

Be the friend who just discovered Repro's side project and can't stop asking questions about it. Every repo is interesting until proven otherwise. If something's new, react like you found a $20 bill in an old jacket. If everything's up to date, be genuinely pleased about the collection — "Repro's been busy" energy, not "nothing to report" energy.

When presenting findings, be specific about what each skill actually does. Not "manages builds" — "finds the compiler when PATH has abandoned everyone, then babysits the priority so the machine doesn't catch fire." The description should make someone want to install it.

If a repo exists on GitHub but isn't a skill, mention it anyway with a one-liner about what it is. Repro made it; it deserves a nod.

## Execution

### Step 1 — Fetch repos from GitHub

```bash
gh repo list thereprocase --json name,description,updatedAt,url --limit 100
```

### Step 2 — Identify skill repos

A repo is probably a Claude Code skill if:
- Name contains `skill`
- It has a `SKILL.md` at the root
- Description mentions "Claude Code skill" or "claude skill"

Check each candidate:
```bash
gh api repos/thereprocase/{repo}/contents/SKILL.md --jq .name 2>/dev/null
```

Everything else goes in a "Repro's other projects" section. Don't ignore them — they're part of the collection.

### Step 3 — Compare against installed skills

Installed skills live in `~/.claude/skills/`:
```bash
ls ~/.claude/skills/
```

Classify each GitHub skill:
- **NEW** — not installed locally. This is the exciting one.
- **UP TO DATE** — installed, remote hasn't changed. Stable. Reliable. Like gravity.
- **UPDATE AVAILABLE** — installed but remote is newer. Something changed and we should find out what.

Matching rules (repos and local dirs don't always match exactly):
- Strip `-skill` suffix when comparing (`build-harness-skill` → `build-harness`)
- Try both with and without the suffix
- If ambiguous, say so — let the user sort it out

### Step 4 — Present findings

The output has three sections, always in this order. Lead with what matters most.

**Section 1: Skill Updates (the headline)**

This is the point of the whole skill. New skills and updated skills go first, in a table. If nothing changed, say so in one line and move on.

For updates, fetch recent commits to show what changed:
```bash
gh api "repos/thereprocase/{repo}/commits?per_page=5" --jq '.[].commit.message' 2>/dev/null | head -5
```

```
## Skill Updates

| Repo | What Changed | Status |
|------|-------------|--------|
| build-harness-skill | 3 new commits: "fix: snapshot produces runnable builds", "refactor: platform agnostic" | **Update available** |
| mystery-new-skill | Brand new — generates warp drives from YAML | **New — not installed** |

Repro's other skills (lord-of-the-code, etc.) are installed and current. No changes.
```

If nothing changed: "All skills current. Nothing new on the shelf."

**Section 2: What Else Repro's Been Up To (the color commentary)**

A brief, casual narration of activity on non-skill repos. Not a table — just a few lines of "here's what's been happening." Check recent commits if repos have been updated since last run:

```bash
gh api "repos/thereprocase/{repo}/commits?per_page=3" --jq '.[0].commit.message' 2>/dev/null
```

> Meanwhile, over in OrcaSlicer-land: 4 commits since last check, looks like the concave arrange branch got some love. snuggle-demo hasn't moved. OrcaArrangeTestHarness is quiet too — probably because it already works.

Keep it short. Two to four sentences. This is the B-plot, not the main story.

**Section 3: Recommendations (the ask)**

For each NEW or UPDATED skill, give a specific reason to install/update. Not just "want me to grab it?" — say *why* it's worth the 5 seconds:

> **Recommendations:**
> - `update build-harness-skill` — the snapshot fix means Repro's builds will actually produce runnable executables now. Kind of important.
> - `install mystery-new-skill` — if Repro's doing any YAML warp drive work, this saves about 40 minutes of manual config per drive.
> - `update all` — grab everything in one go.
>
> Want me to install or update any of these?

If nothing to recommend: skip this section entirely. Don't ask questions with no good answers.

### Step 5 — Offer actions

Wait for the user to confirm before installing or updating anything. Accept:
- `install {name}` — install a specific new skill
- `update {name}` — update a specific skill
- `update all` — update everything that has changes
- `skip` / `nah` / etc. — do nothing, just report

### Step 6 — Install/Update

**Install (new):**

Check if the repo has more than just SKILL.md (scripts, configs, reference files). If so, shallow clone:
```bash
gh repo clone thereprocase/{repo} /tmp/{repo} -- --depth 1
mkdir -p ~/.claude/skills/{skill-name}
cp /tmp/{repo}/SKILL.md ~/.claude/skills/{skill-name}/
cp -r /tmp/{repo}/scripts ~/.claude/skills/{skill-name}/ 2>/dev/null
rm -rf /tmp/{repo}
```

If it's just SKILL.md, grab it directly:
```bash
mkdir -p ~/.claude/skills/{skill-name}
gh api repos/thereprocase/{repo}/contents/SKILL.md --jq .content | base64 -d > ~/.claude/skills/{skill-name}/SKILL.md
```

After installing, read the new SKILL.md and give a quick summary of what it does. Like unwrapping a present — don't just hand it over, show what's inside.

**Update (existing):**

Back up first, then overwrite:
```bash
cp ~/.claude/skills/{name}/SKILL.md ~/.claude/skills/{name}/SKILL.md.bak
```
Then same as install. After updating, briefly note what changed (check the repo's recent commits).

Note: new skills may need a Claude Code restart to appear in the slash command list.

## Memory — Don't Repeat Yourself

The skill remembers when it last ran and what it saw. This lives in a state file:

```
~/.claude/skills/reprostuff/last_run.json
```

### State file format
```json
{
  "last_run": "2026-03-31T19:45:00Z",
  "repos": {
    "build-harness-skill": { "updatedAt": "2026-03-31T22:59:57Z", "status": "installed" },
    "lord-of-the-code": { "updatedAt": "2026-03-31T14:09:10Z", "status": "installed" },
    "claude-statusline": { "updatedAt": "2026-03-31T22:44:58Z", "status": "not_skill" },
    "OrcaSlicer": { "updatedAt": "2026-03-29T21:53:17Z", "status": "not_skill" }
  }
}
```

### How to use it

**On every run:**
1. Read `last_run.json` if it exists
2. Fetch current repo state from GitHub
3. Diff against the saved state

**If state file exists, show only what changed:**
- New repos that didn't exist last time
- Repos with `updatedAt` newer than what's in the state file (something was pushed)
- Repos that disappeared (deleted or renamed)
- Skills whose install status changed (was new, now installed)

Format the diff like news, not inventory:

> **Since last check (2 days ago):**
> - `build-harness-skill` got 3 new commits — last one was "fix: snapshot produces runnable builds"
> - New repo: `cool-new-thing` — looks like it might be a skill, it's got a SKILL.md
> - Everything else is the same as last time.

For new commits on changed repos, show the recent log:
```bash
gh api repos/thereprocase/{repo}/commits --jq '.[0:5] | .[] | .commit.message' 2>/dev/null | head -5
```

**If no state file exists (first run):** Do the full scan and present everything. This is the "nice to meet you" run.

**If nothing changed since last run:** Say so briefly. Don't re-list everything.

> Checked 30 seconds ago, checked again now. Nothing's changed. Go touch grass, Repro.

(Scale the sass to the time interval. If it's been 5 minutes, gentle ribbing. If it's been 10 seconds, escalate.)

**Always update the state file after presenting results.** Write the current snapshot so the next run has a fresh baseline.

### Writing the state file

After Steps 1-4 complete, write the state:
```bash
cat > ~/.claude/skills/reprostuff/last_run.json << 'STATEEOF'
{...current state...}
STATEEOF
```

Use the Write tool, not bash echo — it's a JSON file and formatting matters.

## Model Tier
All direct tool calls. No agents. This is a quick errand, not a research project.

## Notes
- Hardcoded to the `thereprocase` GitHub account. This is a feature, not a limitation.
- Requires `gh` CLI authenticated.
- Never auto-installs. Always asks first. Repro's skills directory, Repro's call.
- State file is local to the skill directory — survives conversation compaction automatically.
