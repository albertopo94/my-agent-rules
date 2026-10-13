# Rotation, safe storage, and leak response

Worked snippets. Every value is a `<PLACEHOLDER>`; substitute from the project, never from a
plausible-looking fake. Nothing in this file, or in the commit that records it, ever holds a real
secret — the commands are written so the value never reaches the command line.

---

## 1. Storing a value without putting it on the command line

**Risk.** `--secret-string "<VALUE>"` and `password=<VALUE>` put the value in the process list and in
shell history, where it survives deletion of the secret itself.

Capture it once, into a variable that never gets echoed, or into a `0600` temp file the tool reads:

```bash
umask 077
read -rs -p "<LABEL>: " SECRET && printf '%s' "$SECRET" > ./<TMP_FILE> && unset SECRET
```

AWS Secrets Manager — `file://` makes the CLI read the payload, not the argument list:

```bash
aws secretsmanager create-secret \
  --name "<PATH/TO/SECRET>" \
  --secret-string file://./<TMP_FILE> \
  --region "<REGION>"
```

Vault — a value prefixed with `@` is read from a file, so it never enters the command line:

```bash
vault kv put "secret/<APP>/<ENV>" username="<USERNAME>" password=@./<TMP_FILE>
```

Treat the temp file as the secret: delete it when the command returns (`rm -f ./<TMP_FILE>`), and
never create it inside the repository working tree.

In scripts, prefer the tool's own stdin or file input over exporting the value into a shell string.
If a CLI accepts only an argument, pass it through a variable the shell expanded from a file — the
history still records the variable name, not the value.

## 2. GitHub Actions: environment secrets and workload identity

Fork PRs receive no repository secrets, so keep every secret-bearing job off untrusted triggers
(see `skills/github-actions-templates`). `id-token: write` is what mints the OIDC token — no static
cloud key in repository secrets.

```yaml
deploy:
  runs-on: ubuntu-latest
  environment: production            # environment-scoped secrets, required reviewers
  permissions:
    contents: read
    id-token: write                  # OIDC federation; without this the job has no identity
  steps:
    - uses: aws-actions/configure-aws-credentials@<FULL_SHA>  # <TAG>
      with:
        role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/<GITHUB_OIDC_ROLE>
        aws-region: <REGION>
        role-session-name: gh-${{ github.run_id }}

    - name: Fetch application secret
      run: |
        value="$(aws secretsmanager get-secret-value \
          --secret-id "<PATH/TO/SECRET>" --query SecretString --output text)"
        echo "::add-mask::$value"     # mask before anything can echo it
        echo "APP_SECRET=$value" >> "$GITHUB_ENV"
```

Vault instead of a cloud store: authenticate with the runner's OIDC token through Vault's JWT auth
method. A `token:` input holding a long-lived `VAULT_TOKEN` is the deviation this file exists to
name; if you must use it, say so in the PR body.

```yaml
    - uses: hashicorp/vault-action@<FULL_SHA>  # <TAG>
      with:
        url: <VAULT_URL>
        method: jwt                  # Vault JWT/OIDC auth method; no static token
        secrets: |
          secret/data/<APP>/<ENV> username | DB_USER ;
          secret/data/<APP>/<ENV> password | DB_PASSWORD
```

Verify the action version's input names against its README before committing; the JWT path needs the
Vault role and the GitHub entity/subject claim configured server-side, which is not a repo change.

## 3. GitLab CI: protected and masked, checked rather than assumed

Masking is **opt-in per variable** in Settings → CI/CD, and the runner only masks a value it can
represent. Setting the flag is not enough — confirm it in the job log.

- [ ] **Protected** on: the variable reaches only protected branches and tags, matching the deploy
      gate.
- [ ] **Masked** on: the value is redacted in job logs. Masking needs a single-line value; older
      runners require base64-safe characters (`A-Za-z0-9+/=_-`) and reject or silently skip masking
      otherwise. Store the base64 form and decode at use.
- [ ] **Type: File** when the consumer wants a path (kubeconfig, key material), not an env var.
- [ ] Scope set per environment/project, not "expose for everything", so a fork pipeline cannot see it.
- [ ] The job never echoes the variable: no `echo "Deploying with $API_KEY"`, no `set -x` in a step
      that holds secrets. Print the variable *name* and where it came from.

## 4. Terraform: keep credentials out of the module and out of state

**Risk.** Any value a resource or data source writes into state is stored **in plaintext**, whatever
`sensitive = true` says. `sensitive` suppresses CLI output only; the state file, its backups, and
anyone who can read the backend still hold the credential.

Provider credentials come from the environment at plan time, never from a `.tf` file or a committed
`terraform.tfvars`:

```bash
export AWS_PROFILE="<PROFILE>"        # or the assumed role the CI identity provides
terraform -chdir="<MODULE_DIR>" plan -out="<PLAN_FILE>"
```

For a database password, do not put it in `aws_db_instance.password` and do not read it into a
`data` source. Pass it out-of-band: create the instance with `manage_master_user_password = true`
where the provider supports it, or hold the credential in the secrets store and have the app read it
at boot, with Terraform managing only the store's name and policy.

Remote state is the highest-value file in the repository: encrypted at rest (`encrypt = true` / the
backend's KMS key), access limited to the pipeline role, and locking enabled so concurrent applies
cannot overwrite it.

## 5. Rotation with an overlap window

**Risk.** Rotating "generate new → revoke old" in one step breaks every consumer that cached the old
value, and clients that refresh on a schedule pick up the new one late.

Two-step rotation: issue the new value alongside the old, let consumers pick it up, revoke the old
only after the overlap window has elapsed and you have seen every consumer authenticate with the new
value. Overlap length = the slowest consumer's refresh interval, plus margin.

```text
rotate(name, overlap):
  old   = read(name)
  new   = generate()
  write(name, { current: new, previous: old, rotated_at: now })
  wait(until max(refresh_interval(consumers)) OR all consumers seen on new)
  verify(no auth events with previous after rotated_at + overlap)
  revoke(old)
  record(name, rotated_at, actor, next_due)
```

AWS's built-in rotation (`aws secretsmanager rotate-secret` with the provider's
`<RDS|Redshift|DocumentDB>` rotation Lambda) drives exactly this sequence against the database and
the secret's `AWSCURRENT`/`AWSPREVIOUS` versions; anything else — an API key, a signing seed — needs
your own rotation job. Record the trigger (scheduled event, due date, or incident) and the human who
owns it in the project docs: an unowned rotation plan does not happen.

## 6. Leaked-secret runbook

Do these in order. Steps 1-2 come before any cleanup, because a secret you can still see is a secret
the attacker can still use.

1. **Revoke or rotate first.** Invalidate the credential at the issuer, now. Cleanup can wait; use
   cannot.
2. **Assume the history is permanent.** A pushed commit is readable by anyone with repository access
   and by whoever mirrored it. Rotating the value is the fix; rewriting history is hygiene, not a
   fix (and never rewrite shared history without the human's decision).
3. **Invalidate derived material.** Sessions, refresh tokens, signed JWTs, issued STS credentials,
   webhook secrets, subkeys, and anything cached from the leaked value — including copies in CI
   caches, artifacts, and logs.
4. **Scan the history** for the same class of secret, not just this one: `gitleaks git .` or
   `trufflehog git file://. --only-verified --fail`, pinned to a version (§7). Scan every branch and
   fork you control.
5. **Scrub and force-rotate everything reachable.** Remove the value from files and logs, then rotate
   every credential the leaked one could derive or reach: same account, same vault path, same
   service account.
6. **Audit the exposure window.** Pull provider audit logs from the earliest plausible commit time to
   revocation and look for use of the credential: unfamiliar source IPs, new API keys, new trust
   relationships, unexpected data reads.
7. **Record the incident.** What leaked, where, how long it was exposed, what was revoked, what the
   scan found, and which control failed — plus the change that stops the same class recurring
   (a scanning hook, a narrower CI scope, a shorter-lived token).

## 7. Scanning hooks

**Risk.** A scanner that runs the code of an untrusted pull request, with secrets in its environment,
hands the attacker a credential-stealer. Scan untrusted code with no secrets and no write token.

Pre-commit, pinned to a tag, staged changes only:

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/gitleaks/gitleaks
  rev: v<GITLEAKS_VERSION>
  hooks:
    - id: gitleaks        # the hook's own entrypoint scans the staged diff
```

Writing the hook by hand instead: run the pinned container over the repo and let a finding exit
non-zero — `docker run --rm -v "$PWD:/repo" ghcr.io/gitleaks/gitleaks:v<GITLEAKS_VERSION> git /repo
--redact`. Subcommands moved across versions (`protect` before 8.19, `detect` as the older alias, `git`
/ `dir` in current releases): check `gitleaks --help` for the version you pinned instead of trusting
this line.

CI gate. It has to be able to fail the pipeline: a scan job that cannot block a merge is decoration.

```yaml
# .gitlab-ci.yml
secret-scan:
  stage: security
  image: ghcr.io/gitleaks/gitleaks:v<GITLEAKS_VERSION>
  variables:
    GIT_DEPTH: "0"                     # full history, or the scan proves nothing about old commits
  script:
    - gitleaks git . --redact
  allow_failure: false                 # never true on a secret scan
```

The GitHub equivalent is a `permissions: contents: read` job that checks out with `fetch-depth: 0`
and runs the same container; a non-zero exit fails the job, so there is no `allow_failure` to
mis-set. Pin the image tag, not `latest`.

Scanning is a backstop for the rules in `SKILL.md`, not a substitute: it finds values that reached a
commit, and it cannot see a value that only ever lived in a command line or a shell history.
