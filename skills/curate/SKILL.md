---
name: curate
description: Score, merge, and prune the user's Claude Code skill library and pending Second Nature proposals — a curator pass that keeps the library sharp. Use when the user says "curate", "clean up my skills", "skill audit", or on a weekly schedule.
---

# Curate — keep the skill library sharp

Audit every skill the user has, score it, and propose keep / update / merge / retire actions. Present the report first; **apply only what the user approves**. Nothing is ever deleted — retired skills are archived.

## Steps

### 1. Inventory

Collect from three places:

- User skills: `~/.claude/skills/*/SKILL.md`
- Project skills: `<project>/.claude/skills/*/SKILL.md` for the current project
- Pending proposals: `~/.claude/second-nature/proposals/*/`

Skip skills that belong to installed plugins/marketplaces (they update upstream; flag them only if broken). Identify plugin-owned skills by their path — only audit skills that live directly in the directories above.

### 2. Score each skill

For each, assess:

- **Usage** — is it ever invoked? Search recent transcripts (`~/.claude/projects/*/*.jsonl`, last 60 days) for its name. Extract invocations cheaply with grep; do not read transcripts whole.
- **Staleness** — do file paths, commands, tools, and URLs it references still exist? Check the load-bearing ones.
- **Overlap** — does another skill cover the same ground? Candidates to merge.
- **Quality** — vague descriptions (won't trigger reliably), missing trigger phrases, instructions that have drifted from how the user actually works now.

For pending proposals: flag any older than 30 days as promote-or-drop decisions.

### 3. Report, then act on approval

Present one table: skill, verdict (keep / update / merge / retire / promote-or-drop), and a one-line reason grounded in the evidence above. Recommendations only — then ask the user what to apply (they can approve all, some, or none).

For approved actions:

- **Retire** — move the skill directory to `~/.claude/second-nature/retired/<YYYY-MM-DD>-<name>/`. Never `rm`.
- **Merge** — write the combined skill, retire the absorbed one (same archive rule).
- **Update** — edit the SKILL.md in place; keep the user's voice and format.
- **Drop proposal** — move it to `retired/` the same way.

Append every applied action to `~/.claude/second-nature/LOG.md` (`YYYY-MM-DD curate: retired <name> — <reason>`).

### 4. Close

Summarize what changed and what was left alone. If the library is healthy, say so — a no-op curate is a valid result. Suggest (once, not naggingly) scheduling `/harvest` + `/curate` weekly if the user hasn't.
