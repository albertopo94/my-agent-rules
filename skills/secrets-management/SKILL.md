---
name: secrets-management
description: "Trigger: secrets in CI or app config, credential storage, rotation, leak response, secret scanning. Decide where a project's credentials live and how they rotate, without hardcoding them or silently introducing a secrets tool."
license: MIT
metadata:
  author: https://github.com/albertopo94
  version: "1.0"
  provenance: "Topic catalog informed by wshobson/agents plugins/cicd-automation/skills/secrets-management @ 156b7a5e (2026-09-29); its examples contained leaked-secret and static-token anti-patterns, rewritten here. Authored in this repo, not vendored."
---

## Activation Contract

Use when a project has credentials to place, move, rotate, or clean up: API keys, database
passwords, deploy tokens, signing keys — wherever they live today (source, `.env`, CI variables, a
vault, a cloud console). Do not use for platform procurement, an HSM/KMS key hierarchy, or end-user
password authentication.

## Hard Rules

- **Existing mechanism first.** Use the store the project declares. If it has none, propose one with
  tradeoffs and **wait for the human decision** — never install Vault, ASM, or KES inside a change.
- **A secret never appears in a command line, a log, or debug output.** Command lines land in shell
  history; pass values by stdin, `file://`, or `@file`. Mask at the CI layer (GitHub `::add-mask::`,
  GitLab Masked variables). Print names and sources, not values.
- **CI authenticates to clouds and vaults by workload identity** — OIDC, JWT, or AppRole with a
  per-run secret ID. A static token in a repository secret is a deviation; name it.
- **Local `vault server -dev` is throwaway testing only:** dev mode, dev token, in-memory. Never
  scripted into a file, never in CI, never holding a real credential.
- **What a secrets backend stores is plaintext to whoever reads the state file.** In Terraform,
  `sensitive = true` hides CLI output only: keep DB credentials out of resources, inject them at plan
  time via provider env vars, keep remote state encrypted with access locked.
- **Scope per environment.** GitHub environment secrets; GitLab protected + masked variables. A job
  gets the named secrets it needs, never `secrets: inherit` from an untrusted caller.
- **Rotation needs an overlap window.** Old and new are both valid until every consumer has moved —
  the secret *and* its clients — or rotation is an outage.
- **Every secret gets an owner, a consumer, and a documented source**; write the mechanism in the
  project README or AGENTS profile area.

## Decision Gates

| Situation | Action |
| --- | --- |
| Single developer, local tool | Process env, or `.env` gitignored plus a committed `.env.example` with keys and no values. No tool |
| Single-cloud team | Cloud-native store first: GitHub environment secrets plus OIDC to ASM / GSM / Key Vault, before any third party |
| Multi-cloud, hybrid, on-prem, or dynamic DB credentials | Centralized store (Vault, external-secrets). New mechanism: propose it, wait for the human decision |
| Secret already leaked | Rotation runbook **first** (`references/rotation-and-scanning.md` §6); scanning and history cleanup after |
| Value differs per environment | One name per environment scope, not one secret plus a branch check |
| Consumer is a human laptop | Short-lived personal credential; never share the CI credential |

## Execution Steps

1. **Inventory.** One row per secret: name, environment, consumer, current source, owner. Read
   `.env.example`, CI config, and deploy scripts — without printing a value.
2. **Pick the mechanism** through the gates. If it is new, write the proposal (cost, operator, what
   breaks without it) and stop for the human decision.
3. **Store the values** with the no-command-line patterns (`references/rotation-and-scanning.md` §1).
   Report the secret *name*, never what you stored.
4. **Wire CI** on workload identity (§2, §3): OIDC role assumption or JWT auth, environment or
   protected scope, masking wherever a step could echo.
5. **Write the rotation plan**: cadence, trigger, new-value path, consumer reload, overlap, revocation.
6. **Add scanning** (§7) as a pre-commit hook plus a blocking CI job, then record the mechanism,
   secret names, and rotation owner in the docs.

## Output Contract

Return:
- The mechanism chosen and why, or the proposal awaiting the human decision.
- The secret inventory: name, environment, consumer, source, owner.
- Per-CI wiring: which job reads which named secret, at what scope, how it authenticates.
- The rotation plan, overlap stated in time, with the reload path per consumer.
- The scanning hook installed, with command and result line; docs updated with mechanism and owner.
- Zero secret values in the diff, in logs, or in this report.

## References

- `references/rotation-and-scanning.md` — safe store/retrieve snippets, CI wiring, rotation with
  overlap, the 7-step leaked-secret runbook, scanning hooks.
- `../../AGENTS.md` §2 (Secrets) — existing mechanism first; mask values when echoing configuration.
- `../../AGENTS.md` §8 (Security checklist) — secrets absent from git history, least-privilege access.
