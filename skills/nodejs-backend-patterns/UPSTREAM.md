# nodejs-backend-patterns — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/wshobson/agents |
| Path in upstream | `plugins/javascript-typescript/skills/nodejs-backend-patterns` |
| Last upstream commit for **this path** | `51b6e0b507aa7942e36c6d33a94fc63aefc53d07` |
| Last modified upstream | 2026-09-28 |
| Fetched here | 2026-10-12 |
| Upstream license | MIT |
| Modified locally | no |

## Check for updates

```bash
gh api "repos/wshobson/agents/commits?path=plugins/javascript-typescript/skills/nodejs-backend-patterns&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.