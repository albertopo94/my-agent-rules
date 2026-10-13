---
name: github-actions-templates
description: Author and review GitHub Actions workflows with security hardening built in. Use when creating CI/CD workflows, reusable workflows, or deployment gates, or when reviewing a workflow for token permissions, injection and supply-chain risks.
license: MIT
metadata:
  author: https://github.com/albertopo94
  version: "1.0"
---

## Activation Contract

Use when creating or editing anything under `.github/workflows/` in a GitHub-hosted repository: new CI,
build, scan, or deploy workflows, reusable workflows, or a security review of an existing one.

Do not use for GitLab CI, Jenkins, or any other runner: the risk classes carry over, the syntax and
trust model do not.

## Hard Rules

- **Default to least privilege.** Every workflow carries an explicit top-level `permissions:` block
  starting from `contents: read`; grant more only per job that needs it.
- **Never interpolate untrusted context into a `run:` block.** `${{ github.event.* }}` values (PR
  titles, branch names, commit messages) go through `env:` with shell quoting — GitHub Actions' #1
  script-injection class.
- **Pin third-party actions to a full commit SHA**, tag in a trailing comment. First-party
  `actions/*` may use majors; use the current major at generation time, not one from an old template.
- **Cloud credentials go through OIDC federation** (short-lived role assumption). Long-lived keys in
  repository secrets are a fallback; name the deviation.
- **`pull_request_target` and `workflow_run` never check out or run untrusted PR code** without an
  explicit reviewed decision.
- **Validate with `actionlint`** before committing a workflow.
- **Every placeholder you emit is `<SNAKE_CASE>`** and is listed for the human to fill. A workflow
  that still contains a placeholder is never "done".

## Decision Gates

| Situation | Action |
| --- | --- |
| Trigger is `pull_request` | Keep it read-only: fork PRs get a read-only token, so no write-capable job runs on it |
| A job needs write scopes (publish, deploy, comment) | Trigger on `push` to a protected branch, or gate the job behind a reviewed approval |
| Production deploy | `environment:` with required reviewers, plus a `concurrency:` group that serializes deploys |
| Matrix build | Only for runtimes the project states it supports (`engines`, docs); otherwise one version |
| Reusable workflow | `secrets: inherit` only between same-org trusted repos; otherwise named secrets |
| Needs a region, cluster, app name, or URL | Read it from the project, else emit `<PLACEHOLDER>` and report it |

## Execution Steps

1. **Read the project before writing YAML.** Trigger, runtime versions from the manifest, secrets
   and OIDC setup, registry, deploy target. Never invent regions, clusters, app names, or URLs.
2. **Pick the pattern** in `references/patterns.md` matching the trigger and the artifact; adapt it.
3. **Harden it** against `references/security-hardening.md`: permissions, untrusted interpolation,
   action SHAs, secret scope, cache and artifact surfaces.
4. **Validate:** `actionlint .github/workflows/<file>.yml`. Fix what it reports, quote the result line.
5. **Report** the placeholder list, every secret the workflow needs by name, and the scopes jobs use.

## Output Contract

Return:
- Workflow files under `.github/workflows/`, each with an explicit top-level `permissions:` block and
  no unguarded untrusted-context interpolation in any `run:` block.
- The placeholder list: one line per token, the value it needs, where to read it.
- The secrets list: name, scope (repository or environment), consuming job.
- The `actionlint` result line.

## References

- `references/patterns.md` — five sanitized shapes: test, image build, reusable workflow, scan, deploy.
- `references/security-hardening.md` — risk-to-fix items and the 10-point reviewer checklist.
- `../../AGENTS.md` §1 (Before writing) — pinning and supply-chain norms apply to actions as to packages.
- `../../AGENTS.md` §8 (Security checklist) — secrets, access, and untrusted-text rules a workflow must
  not bypass.