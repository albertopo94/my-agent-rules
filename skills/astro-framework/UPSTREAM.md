# astro-framework — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/delineas/astro-framework-agents |
| Path in upstream | `skills/astro-framework` |
| Last upstream commit for **this path** | `ba74cc812849c224cb403711fcf0f54a6d81524b` |
| Last modified upstream | 2026-03-22 |
| Repo HEAD when copied | `3e32c91cc433424e162472b81afaa5e9324669bb` |
| Fetched here | 2026-10-13 |
| Upstream license | MIT (`LICENSE.txt`) |
| Modified locally | **yes** — see below |

## Local modifications

1. **`rules/` (9 files) deleted.** Those files use Cursor/Copilot rule frontmatter
   (`description` + `globs`), which is a different mechanism from this runtime's skills and is
   never loaded here. They are also a subset of `references/` — 36 KB of rules versus 132 KB of
   references covering the same topics (`astro-images.rule.md` is 151 lines,
   `references/images.md` is 381).
2. **The `## Guidelines by Context` section removed from `SKILL.md`**, because every entry in it
   pointed at a deleted `rules/*.rule.md` file. Leaving it would have been a pointer to nothing.
3. **`AGENTS.md` (5.8 KB cheatsheet) not copied.** It is a standalone cheatsheet that does not add
   anything to `SKILL.md`'s own Quick Reference section.
4. **`## Quick Reference` gained three blocks merged from the mirror copy** of the same-named
   `astro` skill in `midudev/la-velada-web-oficial` (that copy's own content, MIT; its
   `authors: "Astro Team"` attribution was false and was not carried over): the `npx astro`
   CLI command table, the Deploy Gate (check → build → verify output non-empty), and the
   `docs.astro.build/llms.txt` pointer. Upstream `delineas` has none of these (verified: zero
   occurrences of `astro check`, `astro dev`, `llms.txt` before the merge). They add no
   conflicting instructions, so keep them on update.

Everything else is byte-identical to the upstream revision above. When updating: reapply these
three deletions, because upstream will keep shipping `rules/` and the section that points at it.

## Check for updates

```bash
gh api "repos/delineas/astro-framework-agents/commits?path=skills/astro-framework&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.

## Not vendored from this repository

`skills/learning-astro/` is a multi-file **tutorial** (`guides/`, `concepts/`, `reflect/`,
`help/`). It teaches the framework to a human, which is not a skill contract, so it was left
behind.
