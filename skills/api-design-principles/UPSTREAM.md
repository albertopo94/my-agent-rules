# api-design-principles — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/wshobson/agents |
| Path in upstream | `plugins/backend-development/skills/api-design-principles` |
| Last upstream commit for **this path** | `be57c0b2e3c05c528ca6132b87410b385718775f` |
| Last modified upstream | 2026-05-22 |
| Fetched here | 2026-10-13 |
| Upstream license | MIT |
| Modified locally | no |

## Check for updates

```bash
gh api "repos/wshobson/agents/commits?path=plugins/backend-development/skills/api-design-principles&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.

## Notes on this copy

Fetched from upstream, **not** from the local copy on this machine. The local copy was a stale
pre-split revision: a single 528-line `SKILL.md` with the patterns inline, and no `details.md`.
Upstream restructured it into a 110-line navigation `SKILL.md` plus `references/details.md`. Its
other four files were byte-identical to upstream, which is how the staleness was pinned to the
`SKILL.md` alone rather than guessed.

Two observations, neither acted on, so the copy stays byte-identical and future diffs stay trivial:

- `SKILL.md` links only `references/details.md`. The other four files are reached by convention
  (the loader lists the folder), the same layout as `bash-defensive-patterns` and
  `nodejs-backend-patterns`.
- `assets/rest-api-template.py` is a 182-line FastAPI example inside a stack-agnostic shell. Kept:
  it is an illustrative asset, not an instruction to install Python, and dropping it would make the
  next update check a three-way merge.

Overlap with `nodejs-backend-patterns` is by trigger, not by layer: that skill covers Express and
Fastify implementation, this one covers the API contract — HTTP semantics, versioning strategy,
schema-first GraphQL.
