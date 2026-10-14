# my-agent-rules

My personal **agent shell**: the rules and skills I copy into every new project instead of
re-explaining behavior to a coding agent from scratch each time.

It is not a library and it ships no code. Its output is a `AGENTS.md` that an agent follows on
day one, plus a `Project Profile` that keeps project-specific facts out of the rules so a copy
never carries stale commands from the previous project.

## Quick path

Start a new project:

1. Copy `AGENTS.md` (and `skills/` if you want them) into the project root.
2. Fill the `## 0. Project Profile` table — the only part meant to change per project.
3. Run the port check: it must return nothing.

```bash
grep -rniE 'bridge|pyproject|24980|unittest|python3|README\.md' AGENTS.md
```

Anything that comes back is a leftover fact from another project. Until step 2 is done the agent
treats those fields as declared unknowns, which is safe but noisier than filling them.

## What it changes day to day

| Pain before | What the shell does |
|---|---|
| Every new project starts with "no, don't touch that file" | Rule 1 (Before writing): smallest correct diff, one task at a time |
| "Did you actually run the tests or just assume?" | Rule 4 (Testing) requires the command and its result line, or an `unverified` label |
| Version left stale, or bumped wrong, or bumped and never told me | Rule 3 (Versioning) decides from the commit type, bumps once per work unit, always reports `before → after` |
| Agent invents a plausible command, it fails, then it retries until the error looks like its own fault | The Project Profile turns unknowns into declared blanks; rule 4 forbids swapping commands to explain a failure away |
| Half the rules are things the model already does, so the real rules get less attention | The file is norms only; obvious instructions were deleted, not diluted |
| Secrets leaking into tests, fixtures, logs, commit output | Rule 2 (Secrets) covers all of those surfaces, not just source code |
| A question I did not want asked, or one I did want and never got | Rule 7 (Stop and ask) names the four situations worth stopping for |
| Closing message is a wall of prose | Rule 6 (Closing) fixes the report shape: what, evidence, version, follow-ups |

The measurable effect is fewer interruptions and shorter reviews: the version decision, the
"did you verify" question, and the scope argument stop being part of the conversation.

## How the file is built

Two categories, deliberately never mixed:

- **Norms** (rules 1–7) — hold true in any language, framework, or product shape. Portable.
- **Facts** (the Project Profile, section 0) — this project's test command, version file, platform floor. Edited per project.

The test for any line: *does it still hold if I copy this file into a Go CLI?* If not, it is a
fact and belongs in the Project Profile. That separation is why the shell survives being copied a hundred times:
the parts that go stale are the parts you are required to rewrite.

`Rule 8` is the exception: a security checklist that only applies to network-exposed or
data-persisting products, and is meant to be deleted for a CLI, library, or terminal tool.

## Contents

```
AGENTS.md                        the shell itself
skills/shell-bootstrap/          install the shell in a project, fill the Project Profile, prove no residue
skills/skill-updates/            check vendored skills against upstream, report before touching
skills/UPSTREAM.template.md      provenance record required for any skill copied from elsewhere
LICENSE                          MIT
.gitignore                       stack blocks to prune per project
```

### Vendored skills and updates

A skill folder with an `UPSTREAM.md` came from another repository; one without it was written
here. The record pins the upstream **commit for that skill's own path** plus the revision date and
the fetch date.

Ask an agent *"are the skills current?"* or *"update the skills"* and it loads
`skills/skill-updates/SKILL.md`, which reads each `UPSTREAM.md`, queries upstream for that path
only, and reports `current` / `update available` / `authored here` with a diff before anything is
overwritten. Comparison is against the **recorded commit, never against today**: a repo can be
pushed weekly while one skill sits untouched for a year, so a HEAD-anchored date produces a false
alarm every time.

| Skill | Upstream | Recorded |
|---|---|---|
| accessibility | addyosmani/web-quality-skills | `c6b06ad1` · 2026-08-24 |
| seo | addyosmani/web-quality-skills | `c6b06ad1` · 2026-08-24 |
| frontend-design | anthropics/skills | `41bbe19d` · 2026-09-03 |
| typescript-advanced-types | wshobson/agents | `be57c0b2` · 2026-05-22 |
| nodejs-backend-patterns | wshobson/agents | `51b6e0b5` · 2026-09-28 |
| api-design-principles | wshobson/agents | `be57c0b2` · 2026-05-22 |
| architecture-patterns | wshobson/agents | `be57c0b2` · 2026-05-22 |
| deployment-pipeline-design | wshobson/agents | `be57c0b2` · 2026-05-22 (SKILL.md edited: dead Related Skills removed) |
| bash-defensive-patterns | wshobson/agents | `be57c0b2` · 2026-05-22 |
| astro-framework | delineas/astro-framework-agents | `ba74cc81` · 2026-03-22 |
| cloudflare-agents-sdk | cloudflare/skills | `41e0d198` · 2026-10-01 |
| web-perf | cloudflare/skills | `3af2c774` · 2026-09-05 (MCP example pinned, see UPSTREAM.md) |
| vercel-react-best-practices | vercel-labs/agent-skills | `dc8367e6` · 2026-04-14 (generated AGENTS.md + README + metadata dropped, see UPSTREAM.md) |
| web-design-guidelines | vercel-labs/agent-skills | `ba469388` · 2026-01-16 + rules source pinned, see UPSTREAM.md |
| webapp-testing | anthropics/skills | `b9e19e6f` · 2026-04-20 |
| programmatic-seo | LeoYeAI/openclaw-marketing-skills | `f2d46007` · 2026-03-17 (adopted as authored, aggregator source, scale clause + file write note, see UPSTREAM.md) |
| seo-audit | LeoYeAI/openclaw-marketing-skills | `f2d46007` · 2026-03-17 (adopted as authored, aggregator source, scale clause + file write note, see UPSTREAM.md) |
| programmatic-seo | LeoYeAI/openclaw-marketing-skills | `f2d46007` · 2026-03-17 (adopted as authored, aggregator source, scale clause + file write note, see UPSTREAM.md) |
| agent-e2e-testing, comment-writer, executing-plans, finishing-a-development-branch, github-actions-templates, git-advanced-workflows, interface-design, nodejs-best-practices, receiving-code-review, secrets-management, shell-bootstrap, skill-updates, systematic-debugging, test-driven-development, verification-before-completion, using-git-worktrees, writing-plans | authored here — no upstream | — |

## Deliberately not here

- **Stack overlays** (`python.md`, `go.md`, …). Facts belong in the Project Profile; a per-stack
  rule that survives the copy test should move into the portable rules instead.
- **Clones of skills I already have globally** — work-unit commits, PRs, issue triage, adversarial
  review live in the agent's skill directories, not in each project.
- **Rules the current models satisfy unprompted.** Every line here earned its place; shorter and
  obeyed beats complete and skimmed.

## Next step

After copying the shell, load `skills/shell-bootstrap/SKILL.md` in the new project and let the
agent fill the Project Profile from the repo's own manifests and CI config, then report the port-check result.

---

## External Skills of Interest

Repos with skills I track but don't vendor here — either too large, too niche, or not used often
enough to justify the maintenance burden. Kept here for quick reference.

| Skill / Repo | Why Not Vendored | When to Use |
|---|---|---|
| [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill) | Large skill (dedicated repo), focused on Cloudflare-specific security auditing; repo already has `web-perf` for general perf audits | Auditing Cloudflare-specific configurations, WAF rules, DDoS settings, API Shield, Zero Trust policies |
| [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Massive skill (dedicated repo, 81k★), requires CLI binary, comprehensive UI/UX design system with 20+ sub-commands (craft, shape, critique, audit, polish, bolder, quieter, etc.); repo already has `frontend-design` and `web-design-guidelines` for general design guidance | Comprehensive UI/UX design system work: full design systems, component libraries, visual hierarchy, motion, accessibility audits, design tokens, ambitious visual effects |
| [wheresryan22/anatomy](https://github.com/wheresryan22/anatomy) | Specialized skill for isometric technical illustrations with WebGL/SVG output; dedicated repo with rendering kit (canvas, WebGL, isometric projection); repo already has `frontend-design` for general design | Creating detailed isometric technical illustrations of machines/systems, interactive SVG/WebGL technical diagrams, educational or documentation visualizations of complex systems |
| [wshobson/agents (plugins)](https://github.com/wshobson/agents/tree/main/plugins) | 49 plugin categories with hundreds of skills (backend, frontend, cloud, ML, security, blockchain, game dev, etc.); too large to vendor wholesale; repo already has 7 skills vendored from here (typescript-advanced-types, nodejs-backend-patterns, api-design-principles, architecture-patterns, deployment-pipeline-design, bash-defensive-patterns, git-advanced-workflows) | When you need a specialized skill for a specific domain: Python, Kubernetes, ML ops, payment processing, incident response, observability, reverse engineering, game development, etc. |

