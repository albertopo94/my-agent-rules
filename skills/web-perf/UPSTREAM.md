# web-perf — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/cloudflare/skills |
| Path in upstream | `skills/web-perf` |
| Last upstream commit for **this path** | `3af2c77450c91af61384887def098fbfd247bbad` |
| Last modified upstream | 2026-09-05 |
| Fetched here | 2026-10-13 |
| Upstream license | MIT (repo license; skill carries none of its own) |
| Modified locally | **yes** — see Local modifications |

## Check for updates

```bash
gh api "repos/cloudflare/skills/commits?path=skills/web-perf&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.

## Local modifications

The fetched copy was byte-identical to the upstream tree at the recorded commit (blob
`b028d9cae41d3af2f59d6a90fcac3c241d6ff437`, verified with `git hash-object`). One edit after
fetching (user decision, 2026-10-13): the Chrome DevTools MCP setup example shipped
`npx -y chrome-devtools-mcp@latest`. Per AGENTS.md §1 (verify the exact name and publisher; no
unpinned installs) the example now carries a `<PINNED_VERSION>` placeholder and the prose names
the version instead of pulling latest. Everything else is upstream verbatim. If upstream later
fixes this on its own, drop our edit and take theirs.

## Notes

Hosted by cloudflare/skills but the content is stack- and vendor-neutral: an audit protocol for
Core Web Vitals via Chrome DevTools MCP (no Cloudflare product appears in it). Its distinctive
parts are the retrieval-over-pre-training rule for metric thresholds and the anti-noise reporting
discipline (verify unused before recommending removal; state 0ms impact as a non-issue). Phase 4
overlaps `skills/accessibility` only at snapshot level; that skill remains the a11y authority.
