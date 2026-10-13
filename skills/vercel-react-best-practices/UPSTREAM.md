# vercel-react-best-practices — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/vercel-labs/agent-skills |
| Path in upstream | `skills/react-best-practices` |
| Name in upstream frontmatter | `vercel-react-best-practices` (upstream's folder name differs from its own `name:`; ours matches the frontmatter, which is what the loader requires) |
| Last upstream commit for **this path** | `dc8367e6f91c022d83361f03c3313fa05e848ee5` |
| Last modified upstream | 2026-04-14 |
| Fetched here | 2026-10-13 |
| Upstream license | MIT (repo license; skill frontmatter declares MIT, `author: vercel`, `version: 1.0.0`) |
| Upstream version | 1.0.0 — Vercel Engineering, January 2026 |
| Modified locally | **yes** — see Local modifications |

## Upstream's own references (from its `metadata.json`, which we do not carry)

`https://react.dev` · `https://nextjs.org` · `https://swr.vercel.app` ·
`https://github.com/shuding/better-all` · `https://github.com/isaacs/node-lru-cache` ·
`https://vercel.com/blog/how-we-optimized-package-imports-in-next-js` ·
`https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast`

## Check for updates

```bash
gh api "repos/vercel-labs/agent-skills/commits?path=skills/react-best-practices&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first. Upstream keeps one file per rule,
so an update is usually an added or edited `rules/<file>.md`, not a merge conflict.

## Local modifications

Fetched from upstream. Of its 76 files, 3 were dropped and 1 line edited; the remaining 72 files
(`rules/*`) are byte-identical to the upstream tree at `main` — every blob re-hashed with
`git hash-object` against the trees API, zero mismatches — and `SKILL.md` was byte-identical at
fetch, then edited by the single line below. All mechanical:

| Change | Why |
|---|---|
| Dropped `AGENTS.md` (108 KB upstream) | It is **generated build output** (upstream's own README lists it as "Compiled output (generated)") duplicating the same rules that already live in `rules/`. It also collides by name with this repository's `AGENTS.md`, the shell's own rules file — a 108 KB React manual must not be what a conventional `AGENTS.md` lookup finds inside a skill folder |
| Dropped `README.md` (3.4 KB) | Documents upstream's build machinery (`src/`, `test-cases.json`) that is not part of the copy, so it documents nothing here |
| Dropped `metadata.json` | Its contents — version, organization, date, abstract, reference links — are recorded in this file instead |
| `SKILL.md` last line | Pointed at the dropped `AGENTS.md`; now points at `rules/`. The only reference to a dropped file anywhere in the skill (verified by grep) |

Kept whole, on purpose: all 70 rules including the obvious ones (72 files in `rules/` counting
upstream's `_sections.md` and `_template.md`). Median rule is 1.4 KB, largest 4.4 KB, and the agent
reads the 7 KB index first, then opens the one rule a task needs, so low-value rules cost disk,
never context — while deleting rules would turn every upstream update into a three-way merge.

## Notes

Stack-specific (React and Next.js), accepted the same way as `astro-framework` and
`cloudflare-agents-sdk`: a framework skill earns its place by filling a gap, not by being generic.
Its distinctive value is the **impact ordering** (waterfalls and bundle size CRITICAL down to
advanced patterns LOW), which is what an agent cannot prioritize on its own, plus RSC-era rules that
are not obvious (per-item promise chaining in nested fetches, no module-level mutable request state,
deriving state during render instead of in an effect).

Complements `skills/web-perf`, does not overlap it: web-perf measures in the browser, this tells the
agent what to write in the code.
