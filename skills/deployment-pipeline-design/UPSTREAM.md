# deployment-pipeline-design — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/wshobson/agents |
| Path in upstream | `plugins/cicd-automation/skills/deployment-pipeline-design` |
| Last upstream commit for **this path** | `be57c0b2e3c05c528ca6132b87410b385718775f` |
| Last modified upstream | 2026-05-22 |
| Fetched here | 2026-10-13 |
| Upstream license | MIT |
| Modified locally | **yes** — see Local modifications |

## Check for updates

```bash
gh api "repos/wshobson/agents/commits?path=plugins/cicd-automation/skills/deployment-pipeline-design&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.

## Local modifications

Removed from `SKILL.md` (one edit, 248 bytes): the trailing `## Related Skills` section. Its three
entries pointed at skills that do not exist in this repository (`github-actions-templates`,
`gitlab-ci-patterns`, `secrets-management`); left in place they would be pointers to nothing — the
same reason the `## Guidelines by Context` section was dropped from `astro-framework`.

`references/details.md` and `references/advanced-strategies.md` are byte-identical to upstream
(`git hash-object` verified against the upstream tree). `SKILL.md` is upstream minus the section
above. The stale pre-split copy of this skill was deleted from `~/.agents/skills/` on 2026-10-13
(explicit per-skill user decision).
