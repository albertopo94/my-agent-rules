# Vendored skill — provenance record

Copy this file to `skills/<skill-name>/UPSTREAM.md` for every skill copied from another
repository. Skills I wrote myself get no `UPSTREAM.md` — its absence means "authored here".

The record exists so an agent can later answer, without being told how: *has this skill changed
upstream, and what would we lose or inherit by updating?*

## Template

```markdown
# <skill-name> — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/<owner>/<repo> |
| Path in upstream | `skills/<name>/` |
| Revision copied | `<full commit sha>` |
| Upstream date of that revision | YYYY-MM-DD |
| Date fetched here | YYYY-MM-DD |
| Upstream license | <SPDX id, e.g. MIT / Apache-2.0> + notice preserved? yes/no |
| Modified locally | no \| yes — list every changed file and why |

## How to check for updates

    # 1. Did upstream touch this path since the revision above?
    gh api "repos/<owner>/<repo>/commits?path=skills/<name>&since=<revision date>&per_page=10" \
      --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + .commit.message'

    # 2. Diff upstream against our copy
    git clone --depth 1 https://github.com/<owner>/<repo> /tmp/<repo>
    diff -ru /tmp/<repo>/skills/<name> skills/<name>

## Report format

Upstream unchanged since `<sha>` → say so, nothing to do.
Upstream changed → list: commits touching the path, whether the change adds or removes
instructions, whether it touches anything security-relevant (new commands, new file writes,
new credentials), our local modifications that would conflict, and a keep/update recommendation.
Never overwrite a vendored skill without showing that diff first.
```

## Rules

- **One record per vendored skill, inside that skill's folder.** No central registry: a registry
  drifts from the files it describes.
- Record the **full commit sha**, never a branch name. A branch moves; the sha is what was actually
  read and is the only thing a later diff can be anchored to.
- Record the upstream date of **that revision**, not the fetch date. They differ whenever the
  upstream moved before we noticed, and the gap is the actual update window.
- If a vendored skill is edited here, set `Modified locally: yes` and list the files. Without it,
  an update looks like it applies cleanly when it silently drops our changes.
- Update check is read-only. Applying an update is a normal change: show the diff, keep the
  provenance record current, commit.
