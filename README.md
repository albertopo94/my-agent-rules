# my-agent-rules

My personal **agent shell**: the rules and skills I copy into every new project instead of
re-explaining behavior to a coding agent from scratch each time.

It is not a library and it ships no code. Its output is a `AGENTS.md` that an agent follows on
day one, plus a `Project Profile` that keeps project-specific facts out of the rules so a copy
never carries stale commands from the previous project.

## Quick path

Start a new project:

1. Copy `AGENTS.md` (and `skills/` if you want them) into the project root.
2. Fill the `## 0. Project Profile` table — the only part meant to change per project.
3. Run the port check: it must return nothing.

```bash
grep -rniE 'bridge|pyproject|24980|unittest|python3|README\.md' AGENTS.md
```

Anything that comes back is a leftover fact from another project. Until step 2 is done the agent
treats those fields as declared unknowns, which is safe but noisier than filling them.

## What it changes day to day

| Pain before | What the shell does |
|---|---|
| Every new project starts with "no, don't touch that file" | §1 smallest-correct-diff and one-task-at-a-time are already binding |
| "Did you actually run the tests or just assume?" | §4 requires the command and its result line, or an `unverified` label |
| Version left stale, or bumped wrong, or bumped and never told me | §3 decides from the commit type, bumps once per work unit, always reports `before → after` |
| Agent invents a plausible command, it fails, then it retries until the error looks like its own fault | §0 turns unknowns into declared blanks; §4 forbids swapping commands to explain a failure away |
| Half the rules are things the model already does, so the real rules get less attention | The file is norms only; obvious instructions were deleted, not diluted |
| Secrets leaking into tests, fixtures, logs, commit output | §2 covers all of those surfaces, not just source code |
| A question I did not want asked, or one I did want and never got | §7 names the four situations worth stopping for |
| Closing message is a wall of prose | §6 fixes the report shape: what, evidence, version, follow-ups |

The measurable effect is fewer interruptions and shorter reviews: the version decision, the
"did you verify" question, and the scope argument stop being part of the conversation.

## How the file is built

Two categories, deliberately never mixed:

- **Norms** (§1–§7) — hold true in any language, framework, or product shape. Portable.
- **Facts** (§0) — this project's test command, version file, platform floor. Edited per project.

The test for any line: *does it still hold if I copy this file into a Go CLI?* If not, it is a
fact and belongs in §0. That separation is why the shell survives being copied a hundred times:
the parts that go stale are the parts you are required to rewrite.

`§8` is the exception: a security checklist that only applies to network-exposed or
data-persisting products, and is meant to be deleted for a CLI, library, or terminal tool.

## Contents

```
AGENTS.md                        the shell itself
skills/shell-bootstrap/          install the shell in a project, fill §0, prove no residue
LICENSE                          MIT
.gitignore                       stack blocks to prune per project
```

Local-only, gitignored on purpose:

```
odd/tasks/                       ODD tracking for work done in this repo — working notes, not
                                 part of what gets copied into a new project
.atl/                            regenerated skill-registry cache
```

## Deliberately not here

- **Stack overlays** (`python.md`, `go.md`, …). Facts belong in §0; a per-stack rule that survives
  the copy test should move into §1–§7 instead.
- **Clones of skills I already have globally** — work-unit commits, PRs, issue triage, adversarial
  review live in the agent's skill directories, not in each project.
- **Rules the current models satisfy unprompted.** Every line here earned its place; shorter and
  obeyed beats complete and skimmed.

## Next step

After copying the shell, load `skills/shell-bootstrap/SKILL.md` in the new project and let the
agent fill §0 from the repo's own manifests and CI config, then report the port-check result.
