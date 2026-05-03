---
name: tetrio-replay-organizer
description: Organize TETR.IO replay files in this repository by reading plain-JSON .ttrm files, removing exact duplicates, generating canonical filenames from matchup and winner metadata, updating the replay index table in README.md, and checking for obviously sensitive data before publishing.
---

# TETR.IO Replay Organizer

Use this skill when the task is to import or clean up `*.ttrm` files for this repository.

## What A `.ttrm` File Looks Like

- These replay files are plain JSON, not a binary container.
- Relevant top-level keys seen in this repo: `id`, `gamemode`, `ts`, `users`, `replay`, `version`.
- `users` entries include fields like `id`, `username`, `avatar_revision`, `banner_revision`, `flags`, and `country`.
- `replay.leaderboard` contains per-player summary rows including `username` and `wins`.
- `ts` is an ISO timestamp. Use UTC in filenames and the README table.

## Canonical Workflow

1. Find candidate replay files with `rg --files -g "*.ttrm"` or an equivalent file listing command.
2. Hash every replay with SHA-256 and treat matching hashes as exact duplicates.
3. Keep exactly one canonical copy of each hash. Remove duplicate copies from the repo unless the user explicitly asks to preserve them elsewhere.
4. Parse each retained replay as JSON.
5. Read player display order from `replay.leaderboard` when present. Use the first two leaderboard usernames for the matchup label.
6. Determine the winner by sorting leaderboard entries by `wins` descending. Use the top row as winner and the second row as runner-up.
7. Convert `ts` to a UTC filename timestamp using `yyyy-MM-ddTHH-mm-ssZ`.
8. Sanitize usernames to lowercase slug form by replacing runs of non `A-Za-z0-9._-` characters with `-`, then trim surrounding `-`.
9. Rename the canonical replay to this exact format:

`<UTC timestamp>_<player1>-vs-<player2>_winner-<winner>_<winnerWins>-<runnerUpWins>.ttrm`

10. Update `README.md` so the replay index matches the files currently committed.

## README Format

Keep this exact table shape unless the user asks for something else:

| File | Played (UTC) | Matchup | Winner | Score |
| --- | --- | --- | --- | --- |

- `File` is the literal canonical filename wrapped in backticks.
- `Played (UTC)` is formatted like `2026-05-03 21:47:35 UTC`.
- `Matchup` is `<player1> vs <player2>` using the unslugged usernames from the replay metadata.
- `Winner` is the winner username.
- `Score` is `<winnerWins>-<runnerUpWins>`.

## Duplicate Rules

- Exact duplicates are hash-based, not name-based and not size-based.
- If two files share date, players, and score but have different hashes, keep both because they are distinct replays.
- If a duplicate staging area exists from a prior run, remove it before publishing unless the user explicitly wants an archive of duplicates.

## Privacy And Publishing Checks

Before publishing, scan replay text for common secret patterns. At minimum, look for:

- AWS access keys
- Google API keys
- GitHub tokens
- OpenAI-style API keys
- Slack tokens
- Bearer tokens
- PEM private key headers
- Email addresses

In this repo, previous inspection found no common key or token patterns, but the files do contain usernames, internal-looking user IDs, timestamps, match stats, and full input/event logs. Treat that as publishable gameplay telemetry, not as credentials.

## Repository Expectations

- Keep canonical replay files at the repository root.
- Keep this skill package in `ai/tetrio-replay-organizer/`.
- Update the root `AGENTS.md` only if the entry point needs to change.
- If you change the workflow, update the harness adapters in `.claude/skills/` and `.opencode/commands/` so they still point at this shared skill.

## Practical Validation

- After changes, verify that `git status --short` only shows the intended file adds, deletes, renames, and README updates.
- Verify that every replay listed in the README exists once in the repo root.
- Verify that no duplicate hash remains among committed `.ttrm` files.
