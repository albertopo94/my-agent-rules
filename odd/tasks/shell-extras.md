# Shell extras — README, bootstrap skill, repo boilerplate

**Goal**: make the shell usable without oral instructions — a future project adopts it by reading
the repo, and the daily value is stated where the user will see it.

**Scope agreed**: only artifacts with real value, no filler. README of use, one bootstrap skill,
LICENSE, extended `.gitignore`.

**Out of scope**: rewriting AGENTS.md, adding more skills than the one that closes a real gap,
CHANGELOG (no releases yet), CI.

## Tasks

| # | Task | Done | Evidence |
|---|---|---|---|
| 1 | Load skill-creator, cognitive-doc-design, skill style guide | yes | `~/.agents/skills` + `docs/skill-style-guide.md` |
| 2 | `README.md`: what the repo is, daily-flow value, adoption path | pending | |
| 3 | `skills/shell-bootstrap/SKILL.md`: install shell, fill Profile, port check | pending | |
| 4 | `LICENSE` and stack-agnostic `.gitignore` | pending | |
| 5 | Verify: port-check clean, frontmatter parses, tree matches README | pending | |

## Decisions

- **One skill, not a set.** `work-unit-commits`, `branch-pr`, `issue-creation`, `judgment-day` and
  `skill-creator` already exist in the user's global skill directories; cloning them here would be
  dead weight. The only real gap is shell adoption itself: nothing today automates copying the
  shell, filling §0, and proving no project facts leaked into the copy.
- Skill format follows `docs/skill-style-guide.md`: Activation Contract, Hard Rules, Decision
  Gates, Execution Steps, Output Contract, References. Target 180–450 tokens.
- **No version bump**: this repo declares no version source in AGENTS.md §0, so the version rule
  reports `not bumped (version source undeclared)` rather than inventing a file.
- English content, US spelling, matching AGENTS.md.

## Status

In progress.
