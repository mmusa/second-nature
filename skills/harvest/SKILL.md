---
name: harvest
description: Mine recent Claude Code session history for repeated work patterns and draft them as skill proposals (never auto-installed). Use when the user says "harvest", "what should become a skill", "find repeated patterns", or on a weekly schedule.
---

# Harvest — turn repeated work into skill proposals

Scan the user's recent session transcripts, find work they keep re-doing or re-explaining, and draft each recurring pattern as a **skill proposal**. Proposals are drafts only — nothing becomes an active skill until the user approves it via `/promote`.

## State directory

All Second Nature state lives in `~/.claude/second-nature/`:

- `proposals/<slug>/SKILL.md` — the drafted skill
- `proposals/<slug>/EVIDENCE.md` — why it was proposed
- `retired/` — skills retired by `/curate`
- `LOG.md` — append-only history of proposals, promotions, retirements

Create these directories on first run.

## Steps

### 1. Collect recent history

Transcripts live in `~/.claude/projects/<encoded-project-path>/*.jsonl` — one directory per project, one file per session, one JSON event per line.

- Default scope: sessions modified in the **last 14 days**, across all projects (or only the current project if the user says so). Find them with `find ~/.claude/projects -name "*.jsonl" -mtime -14`.
- These files can be very large. Do NOT read them whole. Extract only the signal:
  - user prompt text (events whose type marks a user message)
  - shell commands run (Bash tool inputs)
  - skill/slash-command invocations
  A `grep`/`jq` pipeline into a scratch file per session is the right shape. Sample long sessions rather than processing every line.
- If more than ~30 sessions match, prefer the most recent 30 and say so in the report — never silently truncate.

### 2. Detect candidate patterns

A pattern is skill-worthy when at least one of these holds:

- The same multi-step workflow appears in **2+ sessions** (3+ is a strong signal).
- The user re-explains the same instructions or preferences to Claude more than once.
- A fiddly incantation recurs (long command lines, exact flag orders, API call shapes) that was clearly re-derived each time.
- A sequence ends in the user correcting Claude the same way repeatedly.

Ignore: one-offs, patterns already covered by an existing skill (check `~/.claude/skills/` and the project's `.claude/skills/`), and anything that is really a personal fact (that belongs in memory, not a skill — note it in the report instead).

### 3. Draft proposals

For each candidate, write `~/.claude/second-nature/proposals/<slug>/`:

**`SKILL.md`** — a complete, installable skill: frontmatter with `name` (short kebab-case) and `description` (what it does + concrete trigger phrases), then step-by-step instructions written from the observed pattern, generalized so it works next time — not a transcript replay.

**`EVIDENCE.md`** — the case for it: which sessions (session-id + date + project), how many occurrences, a one-line description of each occurrence, and an estimate of what the skill saves (time, tokens, or mistakes avoided).

**Secrets rule (hard):** transcripts may contain API keys, tokens, passwords, and private data. Never copy secret values into a proposal — reference them as placeholders (`$API_KEY`, `<your-token>`). If a pattern is inseparable from private data, skip it and say why.

If a proposal for the same pattern already exists, update its EVIDENCE.md with the new occurrences instead of duplicating.

### 4. Report

Append one line per new/updated proposal to `~/.claude/second-nature/LOG.md` (`YYYY-MM-DD harvest: proposed <slug> (N occurrences)`), then give the user a summary table: proposal name, what it automates, occurrences, sessions spanned. End by reminding them: review and install with `/promote`, nothing is active yet.

If nothing recurred enough to propose, say so plainly — an empty harvest is a valid result, not a failure.
