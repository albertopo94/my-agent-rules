---
name: nodejs-best-practices
description: "Trigger: Node.js architecture decision, framework choice, async patterns, error handling, validation, testing strategy, backend design. Node.js development principles and decision-making. Framework selection, async patterns, security, and architecture. Teaches thinking, not copying."
license: MIT
metadata:
  author: albertmac11
  version: "1.0"
  provenance: "adapted from a public copy whose upstream is untraceable; adopted 2026-10-12. Rewritten into the shell's own skill structure — this file is authored here, not vendored."
---

## Activation Contract

Use when making Node.js architecture decisions: choosing a framework, designing async
patterns, structuring error handling, deciding what to test, or reviewing a backend for
correctness.

Do not use for one-off scripts, for framework-specific syntax (that is the framework's own
documentation), or to generate code from a template.

## Hard Rules

- **Teach thinking, never copy code.** A decision tree or a "when to use X" rule is reusable; a
  snippet is not. If the answer is a piece of code, point at the framework docs instead.
- **State the trade-off, not just the recommendation.** Every "use X" carries a cost (cold start,
  ecosystem, learning curve, team familiarity). Say what it costs.
- **Prefer the standard library and what the project already uses.** A framework is a decision,
  not a default.
- **Validate at boundaries, never trust inputs.** Query params, request bodies, headers, cookies,
  file uploads, and external API responses are all untrusted.
- **Decide what is worth testing and say what is not.** Framework code and trivial getters are not
  worth testing; say so rather than inventing coverage.

## Decision Gates

| Situation | Action |
| --- | --- |
| Choosing a framework | Walk the decision tree: deployment target, performance need, team familiarity, ecosystem |
| Legacy or stable stack | Express is the fallback when nothing else wins on the factors above |
| Edge or serverless | Prefer zero-dependency frameworks with fast cold starts |
| High-performance API | Prefer frameworks that beat the default on throughput |
| Enterprise or team | Prefer structure, DI, and convention over configuration |
| Full-stack with a frontend | Consider the framework's own API routes or RPC |
| Not sure | Pick the simplest option that meets the factors, and say why |

## Execution Steps

1. **Name the deployment target first.** Edge, serverless, container, or long-running VM changes
   the answer more than any other single factor.
2. **Score each candidate on the same four factors:** cold start, throughput, ecosystem, learning
   curve. Do not compare different sets of factors per candidate.
3. **State the cost of the chosen option** in one line before recommending it.
4. **Check for an existing helper** before adding one — duplicating a utility in a second file is a
   defect, not a convenience.
5. **For error handling:** centralize it, choose status codes deliberately, and never leak stack
   traces or internal details to the caller.
6. **For async:** pick the pattern by workload, not by trend. Be aware of what blocks the event
   loop and avoid it.
7. **For testing:** prioritize critical paths (auth, payments, core business), edge cases, and
   error handling. Explicitly name what is not worth testing.

## Output Contract

Return: the decision made, the four factors it was scored on, the cost of that decision, and the
open factors that would change it. For a review: the anti-patterns found, each with the rule that
prevents it.

## References

- `../../AGENTS.md` §1 — read before you write, including the search-for-existing-helper rule.
- `../../AGENTS.md` §4 — scale the check to the change before deciding what to test.