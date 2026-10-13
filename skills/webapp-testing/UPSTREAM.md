# webapp-testing — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/anthropics/skills |
| Path in upstream | `skills/webapp-testing` |
| Last upstream commit for **this path** | `b9e19e6f` (2026-04-20) |
| Fetched here | 2026-10-13 |
| Upstream license | Apache-2.0 (`LICENSE.txt`, complete terms referenced by the skill frontmatter) |
| Modified locally | no — all 6 files byte-identical to the upstream tree (each blob re-hashed with `git hash-object`) |

## Check for updates

```bash
gh api "repos/anthropics/skills/commits?path=skills/webapp-testing&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first. Note that `scripts/` files are
executable behavior, not prose: a change there changes what runs on the developer machine, so diff
those line by line.

## Known limitations, accepted as-is

Recorded so a future update check does not "fix" them silently or lose the reasoning:

- **Requires Python + Playwright on the machine using it** (`pip install playwright`, browser
  download). The skill does not install them and this shell does not declare a stack, so nothing
  here installs them either: in a project without Python, the skill's own instructions degrade to
  writing Playwright scripts the developer runs. That is the same stance as the Project Profile —
  the project declares its toolchain, the shell does not choose it.
- **`scripts/with_server.py` starts server commands with `shell=True`.** Intentional and
  load-bearing: it accepts a developer-provided command (`--server "cd backend && python server.py"`)
  and supports `cd`/`&&` to chain them. It only ever reaches processes it started itself, and its
  `finally` block terminates them even on failure. Treat a change to that file as a change to
  executable behavior.
- **Upstream typo**: SKILL.md says "abslutely". Left alone. Fixing one letter turns every future
  update into a three-way merge for zero behavior change; if upstream fixes it, take theirs.

## Notes

This is the shell's functional-verification leg: AGENTS.md §1 requires shipped UI states and §4
requires evidence over assertion, and `frontend-design` says to take screenshots to review — but
until now nothing in the repo said how to drive a real browser to do it. `agent-e2e-testing` does not
cover this: that skill builds a deterministic suite for an agentic system behind fake
model/MCP boundaries, this one is the ad-hoc browser check of a UI change (screenshot, console logs,
click-through).

Its non-obvious content is discipline rather than API: the decision tree (static HTML → read the
file for selectors; dynamic → is the server up?), the reconnaissance-then-action pattern (wait for
`networkidle`, inspect the **rendered** DOM, derive selectors from it, then act), and the
black-box rule — call `scripts/*` with `--help`, never read their source, so a working helper does
not cost context.
