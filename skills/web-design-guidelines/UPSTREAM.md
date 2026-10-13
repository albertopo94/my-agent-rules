# web-design-guidelines — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/vercel-labs/agent-skills |
| Path in upstream | `skills/web-design-guidelines` |
| Last upstream commit for **this path** | `ba469388` (2026-01-16) |
| Fetched here | 2026-10-13 |
| Upstream license | MIT (repo license; frontmatter `author: vercel`, `version: 1.0.0`) |
| Modified locally | no — `SKILL.md` is byte-identical to upstream (blob `ceae92ab`) |

## The real upstream is a second repository

`SKILL.md` is a 1.2 KB **pointer**: it fetches its rule set at run time from another repo, so the
rules that actually govern a review do not live in this file, and pinning only this file proves
nothing. Pin both.

| Pinned | Value |
|---|---|
| Rules source URL | `https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md` |
| Source repo | `vercel-labs/web-interface-guidelines` (972 stars, MIT) |
| Blob sha of `command.md` at fetch | `3eb3a212b7b89493922ea418489d4c6f67ae8bd5` |
| Last commit touching `command.md` | `4ecfb9fb` (2026-10-05) — the rule set is actively maintained |

Verify the pointer still resolves **and** the rules have not moved:

```bash
# 1. Pointer still upstream?
gh api "repos/vercel-labs/agent-skills/commits?path=skills/web-design-guidelines&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'

# 2. Rules source still there, and changed? (a 404 here means the skill silently degraded)
curl -s https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md \
  | git hash-object --stdin    # compare with 3eb3a212...
gh api "repos/vercel-labs/web-interface-guidelines/commits?path=command.md&per_page=5" \
  --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
```

**If check 2 hashes to anything other than `3eb3a212...`** — including the body `404: Not Found`
that raw.githubusercontent returns for a missing path (it hashes to `1becba2b...`, it is not an
empty response): the upstream moved the file. The skill then fetches
nothing and reviews against whatever the model remembers, which reads as a successful review. Find
the new path in that repository and update both this record and `SKILL.md` — do not leave the dead
URL in place.

## Report

**Unchanged** — pointer commit equals the recorded sha *and* the source blob equals `3eb3a212`: say
so, nothing to do.

**Changed** — show what the rules gained or lost (diff the two `command.md` versions), whether any
new rule conflicts with `skills/accessibility` (which stays the authority on WCAG criteria), and a
keep or update recommendation. The rules are fetched at run time, so an update here changes only
what this record pins, never the skill's behavior.

## Notes

Cost is one 1.2 KB file: the skill ships no rules of its own, it makes the agent pull the current
ones, the same retrieval-over-pre-training stance as `skills/web-perf`.

Where it overlaps `skills/accessibility` (accessibility, focus states) that skill remains the
authority — it carries numbered WCAG 2.2 criteria while this list is terse. What is not covered
anywhere else in the shell: form mechanics (never block paste, clickable labels with a single hit
target, `spellCheck={false}` on emails and codes, submit stays enabled until the request starts),
sticky overlays covering the focused element, `scroll-margin-top` on anchors, safe areas, locale and
i18n, hydration safety, and the anti-pattern list. `../../AGENTS.md` §1 requires the interactive
states this checks for; this is how you verify a UI change against that rule.
