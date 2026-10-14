# git-advanced-workflows — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/wshobson/agents |
| Path in upstream | `plugins/developer-essentials/skills/git-advanced-workflows` |
| Last upstream commit for **this path** | `be57c0b2e3c05c528ca6132b87410b385718775f` |
| Last modified upstream | 2026-05-22 |
| Fetched here | 2026-10-14 |
| Upstream license | MIT |
| Modified locally | no |

## Check for updates

```bash
gh api "repos/wshobson/agents/commits?path=plugins/developer-essentials/skills/git-advanced-workflows&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.

## Notes on this copy

Same pattern as other wshobson skills: upstream restructured the skill into a navigation `SKILL.md`
plus `references/details.md` with all the practical workflows. The PC copy was the stale pre-split
monolith. All 2 files verified byte-identical to the upstream tree at the path-scoped commit
`be57c0b2`. The PC copy was the old monolithic revision — vendorized from upstream instead.