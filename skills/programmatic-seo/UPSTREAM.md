# programmatic-seo — upstream (adopted, aggregator source)

| Field | Value |
|---|---|
| Upstream repository | https://github.com/LeoYeAI/openclaw-marketing-skills |
| Path in upstream | `skills/programmatic-seo` |
| Last upstream commit for **this path** | `f2d46007` (2026-03-17) |
| Fetched here | 2026-10-13 |
| Upstream license | NOASSERTION |
| Modified locally | **yes** — adopted as authored, two local edits (scale clause + project file write note) |

## Provenance

This skill was copied from an aggregator repository (LeoYeAI/openclaw-marketing-skills) which
compiles marketing skills from various sources. The original author is untraceable; the aggregator
itself is not a canonical authority like wshobson/agents or cloudflare/skills. The content is
excellent and complete; we adopted it as authored here to preserve it against the aggregator
disappearing, with the same local edits as `interface-design` (scale clause + project-file note).

## Check for updates

```bash
gh api "repos/LeoYeAI/openclaw-marketing-skills/commits?path=skills/programmatic-seo&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first. If the repo 404s, we already have
the full content and the skill is self-contained.

## Local modifications

Two edits, both recorded here and in frontmatter provenance:

1. **Scale clause** (inserted after `## Core Principles`): the full 12-playbook framework is for a
   new programmatic SEO initiative or major expansion; a bounded change applies only the relevant
   playbook and its quality checks, citing AGENTS.md Rule 4 as the scaling rule.

2. **Project file write note** (inserted in `### Page Template`): if the skill writes page
   templates or data files to the project, name the paths out loud before writing and let the human
   decide versioning. Treat them as project knowledge worth committing, not tool cache.

Both edits match the local edits on `interface-design` and `secrets-management` for consistency.