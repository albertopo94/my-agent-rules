---
name: agent-e2e-testing
description: "Trigger: test agent e2e, agent e2e testing, fake model, fake mcp, sandbox isolation, deterministic agent testing. Deterministic end-to-end testing for agentic systems using fake models, fake MCP servers, sandbox isolation, and config-level dependency injection — no tokens spent, no external APIs called."
license: MIT
metadata:
  author: https://github.com/albertopo94
  version: "1.0"
---

## Activation Contract

Use when setting up or running end-to-end tests for an agentic system (any SDK: Strands, LangGraph, Vercel AI, etc.) and the goal is deterministic, fast, zero-cost tests that exercise the full stack — tools, multi-agent handoffs, streaming, structured output, MCP calls — without calling real LLMs or external services.

Do not use for unit testing individual tools, for load testing, or when the test must verify real model behavior (that is a different test tier).

## Hard Rules

- **Zero tokens, zero external calls.** Every e2e test run must complete without spending tokens or hitting real APIs. If a test needs a real model or service, it is not an e2e test in this sense — it is an integration test and belongs in a separate tier.
- **Fake the contracts, not the internals.** Implement the public interfaces the SDK exposes (`Model`, `MCPClient`, `Tool`, `Agent`) so the application code does not know it is under test. Do not mock private methods or reach into framework internals.
- **Sandbox the world.** Tests run in an isolated temporary directory (`e2e/.sandbox/`) with their own `data/`, credentials, and config. The real `data/`, `.coros/`, `.env` are never touched. Cleanup is automatic.
- **Swap at the config/entry point, not in the code.** A single env var (`E2E_FAKE_AI=1`) or config flag swaps the real `Model`/`MCPClient` for the fakes. No `if (test)` branches in production code.
- **Deterministic seeds, not randomness.** Fake model responses are derived from the prompt and available tools. Same input → same output. Flakiness is a bug in the fake, not in the test.
- **Exercise the full graph.** The test harness launches the real application (dev server, agent runtime, streaming endpoints) against the fakes. The test client (Playwright) talks to real HTTP/WebSocket endpoints. The only fake layer is the model/MCP boundary.

## Decision Gates

| Situation | Action |
|---|---|
| SDK has no injectable `Model` interface | Do not force this pattern; document the limitation and use a higher-tier integration test instead |
| Real MCP server has no discoverable tool schema | The fake MCP server must implement the same tool discovery protocol (listTools, callTool) as the real one |
| Test needs to verify model reasoning quality | Out of scope — this skill is for wiring, orchestration, tool calls, streaming, handoffs; not model quality |
| Multi-agent system (swarm, agent-as-tool) | Fake model must support the same handoff protocol (structured output with `agentId`, `context`) as the real SDK |
| Streaming / NDJSON / SSE | Fake model must emit the same event stream shape (textDelta, toolCall, toolResult, handoff) |

## Execution Steps

1. **Define the fake Model contract.** Implement the SDK's `Model` interface (or abstract class) with a single `generate`/`stream` method that:
   - Receives prompt + available tools + conversation history
   - Returns deterministic text, tool calls, structured output, or swarm handoffs based on keywords in the prompt
   - Emits the same event stream shape as the real model (textDelta, toolCall, toolResult, handoff)

2. **Define the fake MCP server.** Implement the MCP protocol (stdio or HTTP) with:
   - `initialize`, `listTools`, `callTool` endpoints
   - A registry of tool names → deterministic handlers returning fixed or seed-derived data
   - The same auth/transport the real server uses (stdio, SSE, HTTP)

3. **Create the sandbox harness.**
   - Temp directory per test run (`e2e/.sandbox/<run-id>/`)
   - Seed files: fake tokens, pre-baked cache files (races, predictions, briefings), a fake `.env`
   - `cwd` of the launched app = sandbox dir; real `data/`, credentials untouched

4. **Wire the swap at config level.**
   - `astro.config.mjs` / `vite.config.ts` / `next.config.js` reads `process.env.E2E_FAKE_AI`
   - When truthy, exports the fake Model and fake MCP client factory instead of the real ones
   - No `if (test)` in application code; the swap is invisible to the app

5. **Build the test runner.**
   - `e2e/fakes/start-app.ts`: seeds sandbox, launches `astro dev` (or `next dev`, `vite dev`) on a free port (e.g., 4399)
   - Waits for `/health` or a known endpoint before returning
   - Tears down the child process on test suite teardown

6. **Write Playwright tests against the real endpoints.**
   - Tests hit `http://localhost:4399` (or whatever port)
   - Assertions on DOM, API responses, streaming chunks, NDJSON events, WebSocket messages
   - Multi-agent: assert handoff events, shared `invocationState`, swarm steps recorded

## Output Contract

Return:
- A working `pnpm test:e2e` (or `npm run test:e2e`) that runs in <30s on CI
- `e2e/fakes/model.ts` implementing the SDK's `Model` interface
- `e2e/fakes/mcp-server.ts` implementing the MCP protocol
- `e2e/fakes/start-app.ts` sandbox launcher
- Config-level swap (env var + config file change)
- At least one test exercising: tool call → result, structured output, streaming, multi-agent handoff
- CI config that runs the suite on every PR

## References

- `../../AGENTS.md`, Rule 4 (Testing and verification) — evidence, not assertion; tests are evidence.
- `skills/agent-guardrails-patterns` — The fake model should also exercise the guardrail hooks if the app uses them.
- Original pattern source: midudev/strands-midu-demo (`e2e/fakes/`, `astro.config.mjs`, `src/agent/guardrails.ts`).