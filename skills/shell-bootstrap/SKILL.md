---
name: shell-bootstrap
description: "Trigger: bootstrap shell, install AGENTS.md, adopt agent rules, new project shell, fill project profile, port check. Copy the personal agent shell into a project, fill its Project Profile from the repo's own evidence, and prove no foreign project facts leaked in."
license: Apache-2.0
metadata:
  author: https://github.com/albertopo94
  version: "1.0"
---

## Activation Contract

Use when adopting the shell in a project: copying `AGENTS.md`, filling `## 0. Project Profile`, or
checking a copied shell for residue from the project it came from.

Do not use to edit shell norms (rules 1-7), to author skills, or where the project already has its
own `AGENTS.md` — that file wins; propose a merge instead.

## Hard Rules

- Every Profile value comes from **evidence in this repository** (manifest, lockfile, CI config,
  scripts, README) and cites its source file. Never carry a value over from another project.
- An unverifiable field **stays empty** and is reported as undeclared. Never invent a command to
  fill it, and never run a candidate command just to see what it does.
- Only the Project Profile and deleting the Rule 8 security checklist are in scope. Never edit rules 1-7.

## Decision Gates

| Situation | Action |
| --- | --- |
| No `AGENTS.md` in target | Copy the shell, then fill the Project Profile |
| `AGENTS.md` exists, no Project Profile | Append it; do not touch the norms |
| Project Profile partly filled | Fill gaps only; leave existing values untouched |
| Not network-exposed, persists no data | Delete the Rule 8 checklist and say so |
| Network-exposed or persists data | Keep Rule 8; delete only inapplicable bullets |
| Two plausible test runners | Report both, choose the one CI runs, state why |

## Execution Steps

1. Read the evidence: dependency manifest, lockfile, test/lint config, CI workflow, entrypoints.
   Read only — executing is not part of bootstrapping.
2. Fill the Project Profile: value plus source file per field; list the fields left empty explicitly. Derive
   `Stability contract` from the declared version and tags, and `Public surface` from exported
   symbols, CLI flags, routes, or config keys actually present.
3. Apply the Rule 8 gate.
4. Port check — must return nothing:

   ```bash
   grep -rniE 'bridge|pyproject|agy-|24980|unittest|evaluar' AGENTS.md
   ```

5. Refresh the project skill registry when that command exists.

## Output Contract

Return: files created or modified; a `field → value → source` table; fields left undeclared;
whether Rule 8 stayed and why; the port-check command with its real output; open ambiguities.

## References

- `../../AGENTS.md` — the Project Profile is the only mutable section; the Rule 8 checklist is the only deletable one.
- `../../README.md` — why norms and facts stay separate.
