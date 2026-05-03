---
description: Organize TETR.IO replay files in this repository using the shared replay skill
---

Read @ai/tetrio-replay-organizer/SKILL.md and carry out that workflow for the current repository state.

If the user supplied new `.ttrm` files, ingest them.
If duplicates exist, deduplicate by SHA-256.
If filenames are non-canonical, rename them.
If the replay set changed, update @README.md.
Before publishing, do the privacy scan described in the shared skill.
