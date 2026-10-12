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
| 1 | Load skill-creator, cognitive-doc-design, skill style guide | yes | `docs/skill-style-guide.md` section order + token budget applied |
| 2 | `README.md`: what the repo is, daily-flow value, adoption path | yes | commit `7c231d1` |
| 3 | `skills/shell-bootstrap/SKILL.md`: install shell, fill Profile, port check | yes | commit `662899d`; 428 words, ~570 tokens, style-guide section order |
| 4 | `LICENSE` and stack-agnostic `.gitignore` | yes | commit `52b6a7e` |
| 5 | Verify: port-check clean, frontmatter parses, tree matches README | yes | `grep` port-check on AGENTS.md returns empty; frontmatter is one YAML-safe line; `git ls-files` matches README Contents |

## Verification commands

```bash
grep -rniE 'bridge|pyproject|24980|unittest|python3|README\.md' AGENTS.md   # empty
wc -w skills/shell-bootstrap/SKILL.md                                       # 428
```

## Deliberately skipped

- **CHANGELOG** — nothing is released, so it would be an empty file asserting a history.
- **CI** — a docs-only repo with nothing to build or test.
- **A second skill** — no gap left that the global skill set does not already cover.

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

Complete. `version: not bumped (this repo declares no version source in AGENTS.md §0)`.
