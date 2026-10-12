# Vendored skill — provenance record

Copy this file to `skills/<skill-name>/UPSTREAM.md` for every skill copied from another
repository. A skill folder **without** `UPSTREAM.md` was written here.

The record exists so any agent can answer, without being told how: *has this skill changed
upstream since we copied it, and what would updating change for us?*

## License

Not a gate. If the source is a public repository we copy it, and the license is recorded as an
informational field so the origin is traceable. Do not block, debate, or ask about licensing.

## Template

```markdown
# <skill-name> — upstream

| Field | Value |
|---|---|
| Upstream repository | https://github.com/<owner>/<repo> |
| Path in upstream | `skills/<name>` |
| Last upstream commit for **this path** | `<full sha>` |
| Last modified upstream | YYYY-MM-DD  *(date of that path-scoped commit)* |
| Repo HEAD when copied | `<full sha>` |
| Fetched here | YYYY-MM-DD |
| Upstream license | <SPDX id, or "none declared"> |
| Modified locally | no \| yes — list each changed file and why |

## Check for updates

    # Has upstream touched THIS path since the recorded commit?
    gh api "repos/<owner>/<repo>/commits?path=skills/<name>&per_page=5" \
      --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'

    # Anything newer than the recorded sha means an update exists.

    # What changed upstream since our commit, for this path only:
    gh api "repos/<owner>/<repo>/compare/<recorded sha>..HEAD" --jq '.files[] | select(.filename | startswith("skills/<name>")) | .status + "  " + .filename'

    # Full diff against our local copy, including our own edits:
    git clone --depth 1 https://github.com/<owner>/<repo> /tmp/<repo>
    diff -ru /tmp/<repo>/skills/<name> skills/<name>

## Report

**Unchanged** — newest commit for the path equals the recorded sha: say so, nothing to do.

**Changed** — list the commits touching the path, then: what instructions are added or removed,
whether anything security-relevant appears (new commands to run, new files written, credentials or
network calls requested), whether our local edits conflict, and a keep or update recommendation.
Never overwrite a vendored skill without showing that diff first.
```

## Rules

- **One record per vendored skill, inside that skill's folder.** No central registry: it drifts
  from the files it describes.
- **Anchor every date and sha to the skill's path, never to the repository HEAD.** A repo can be
  updated weekly while a given skill sits untouched for a year; a HEAD-anchored date turns every
  check into a false alarm. `Repo HEAD when copied` is recorded only as context for locating the
  snapshot, never as the change indicator.
- Store the **full commit sha**, not a branch name. A branch moves; the sha is what was actually
  read and the only thing a later diff can anchor to.
- If the skill is edited here, set `Modified locally: yes` and list the files — otherwise an update
  looks like it applies cleanly while silently dropping our changes.
- Update checks are read-only. Applying an update is an ordinary change: show the diff, refresh the
  record, commit.
