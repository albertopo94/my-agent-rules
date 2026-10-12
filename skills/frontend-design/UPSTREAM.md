# frontend-design — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/anthropics/skills |
| Path in upstream | `skills/frontend-design` |
| Last upstream commit for **this path** | `41bbe19d1a1a7eaab5e7bb9050a417e5c6cffc8f` |
| Last modified upstream | 2026-09-03 |
| Fetched here | 2026-10-12 |
| Upstream license | MIT, see `LICENSE.txt` |
| Modified locally | no |

## Check for updates

```bash
gh api "repos/anthropics/skills/commits?path=skills/frontend-design&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.