---
name: promote
description: Review pending Second Nature skill proposals and install approved ones as real Claude Code skills — the human approval gate. Use when the user says "promote", "review proposals", "install that skill", or after a harvest.
---

# Promote — the approval gate

Show the user their pending skill proposals and install only the ones they approve. This is the single point where a draft becomes an active skill; it never runs automatically.

## Steps

### 1. List pending proposals

Read `~/.claude/second-nature/proposals/*/`. For each, present: proposed name, what it automates (from its SKILL.md description), and the evidence summary (occurrences, sessions, estimated savings from EVIDENCE.md). If there are none, say so and suggest running `/harvest`.

### 2. Let the user choose

Ask which to install (all / some / none), and for each approved one whether it should be:

- **User-level** → `~/.claude/skills/<name>/` (available in every project) — the default
- **Project-level** → `<project>/.claude/skills/<name>/` (only makes sense if the pattern is specific to one repo)

The user can also request edits before installing — apply them to the proposal first.

### 3. Install

For each approved proposal:

1. **Collision check** — if a skill with that name already exists in the target (or an installed plugin exposes one), stop and ask: overwrite, rename, or merge into the existing skill.
2. Move the proposal directory's `SKILL.md` (and any supporting files, but not EVIDENCE.md) to the target skill directory.
3. Move the remaining proposal folder to `~/.claude/second-nature/retired/<YYYY-MM-DD>-promoted-<name>/` so evidence is preserved but the queue stays clean.
4. Append to `~/.claude/second-nature/LOG.md`: `YYYY-MM-DD promote: installed <name> (user|project)`.

### 4. Close

Confirm what was installed and where, with each skill's trigger word. Note that new skills are picked up by new sessions — in the current session the user can invoke them by name once the file exists.

Declined proposals stay in the queue unless the user says drop them (then archive to `retired/`, never delete).
