---
name: skill-updates
description: "Trigger: update skills, check skill updates, are the skills current, skill upstream, revisar actualizaciones de skills, actualizar skills vendorizadas. Compare every vendored skill in this repository against its upstream and report what changed without overwriting anything."
license: MIT
metadata:
  author: https://github.com/albertopo94
  version: "1.0"
---

## Activation Contract

Use when asked whether vendored skills are current, or to update them.

Do not use for skills authored here (they have no upstream), for the global skill directories
outside this repository, or to update a skill the user has not asked about.

## Hard Rules

- **Read-only until the report.** Never overwrite, delete, or edit a vendored skill before showing
  its diff and getting approval. An approved update is an ordinary change: apply, refresh the
  record, commit.
- **Compare against the recorded commit, never against today.** Each `UPSTREAM.md` pins the last
  upstream commit for that skill's own path. Repository HEAD moving is not evidence that a skill
  changed — a repo can be pushed weekly while a given skill sits untouched for a year.
- **Scope every query to the recorded path**, never to the whole repository.
- **Report skills with no `UPSTREAM.md` as authored here.** They have nothing to check; do not
  invent an upstream for them.
- Flag any upstream change that adds commands to run, files to write, credential or network use —
  those are instructions the agent would execute, not text.

## Execution Steps

1. Enumerate `skills/*/`: each folder with an `UPSTREAM.md` is vendored, each without it is ours.
2. From each record read: upstream repository, path in upstream, recorded commit, local
   modifications.
3. Query upstream for that path only:

   ```bash
   gh api "repos/<owner>/<repo>/commits?path=<path>&per_page=5" \
     --jq '.[] | .sha[0:8] + "  " + .commit.author.date[0:10] + "  " + (.commit.message | split("\n")[0])'
   ```

4. Newest commit equals the recorded sha → unchanged. Newer → list the commits touching the path.
5. For changed skills, show what actually differs before recommending anything:

   ```bash
   gh api "repos/<owner>/<repo>/compare/<recorded sha>..HEAD" \
     --jq '.files[] | select(.filename | startswith("<path>")) | .status + "  " + .filename'
   ```

6. Weigh recorded local modifications against the incoming change; a clean-looking update that
   silently drops our edits is the failure mode to prevent.

## Output Contract

Return one table: skill, recorded date and commit, upstream latest date and commit, status
(`current` / `update available` / `authored here` / `no upstream found`). Then, for each update
available: what instructions are added or removed, anything security-relevant, conflict with our
local edits, and a keep or update recommendation. End with the applied and declined updates.

## References

- `UPSTREAM.template.md` — the field meanings and the update-check contract each record follows.
- `AGENTS.md` §4 — quote the commands actually run; never claim current without evidence.