# agents-sdk — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/cloudflare/skills |
| Path in upstream | `skills/agents-sdk` |
| Last upstream commit for **this path** | `41e0d19858946d18af9ee2c2feebbe2e11d829ff` |
| Last modified upstream | 2026-10-01 |
| Fetched here | 2026-10-13 |
| Upstream license | MIT |
| Modified locally | no |

## Check for updates

```bash
gh api "repos/cloudflare/skills/commits?path=skills/agents-sdk&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.