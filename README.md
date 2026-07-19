# Heyward Hagenbuch

AI/agent engineering on an enterprise JVM stack. I build the unglamorous parts
that make LLM systems shippable: bounded tool-calling loops, MCP integrations,
eval gates in CI, and the reactive plumbing underneath.

Day job: platform & AI tooling for a 60+ microservice fleet (Java/Spring
WebFlux, Kubernetes, GraphQL federation) — including an internal code-intelligence
MCP server with 50+ tools that gives coding agents whole-workspace context.

## The platform

These repos aren't seven demos — they compose. A production-shaped agent core,
and the layers you actually need to run one: observe its cost, record what it did,
survive a dead link, deploy it safely, gate it on evals, and contract-test its tools.

```mermaid
flowchart TB
    starter["<b>spring-ai-agent-starter</b><br/>reactive tool-calling agent core"]

    meter["<b>agent-meter</b><br/>OTel cost attribution + budgets"]
    blackbox["<b>agent-blackbox</b><br/>flight recorder + trace export"]
    castaway["<b>castaway</b><br/>runtime for disconnected environments"]
    operator["<b>agent-operator</b><br/>k8s operator: prompt-version canaries"]
    evals["<b>agent-evals</b><br/>JUnit for agents — CI exit-code gate"]
    mcppact["<b>mcp-pact</b><br/>contract tests for MCP servers"]

    meter -. "wraps LlmClient (cost)" .-> starter
    blackbox -. "wraps LlmClient (record)" .-> starter
    mcppact -->|"guards the MCP tool boundary"| starter
    castaway -->|"reuses the core, adds link-failover"| starter
    operator ==>|"deploys + rolls prompt versions"| starter
    operator -. "canary gate runs" .-> evals
    evals ==>|gates| starter
    evals ==>|gates| castaway
    evals ==>|gates| operator

    classDef core fill:#1f6feb,stroke:#0b3d91,color:#fff;
    class starter core;
```

**Legend:** dotted = non-invasive decorator seam (add a dependency, no code change);
solid = builds on / guards; thick = eval gate.

## Repos

**Core**
- [spring-ai-agent-starter](https://github.com/hhagenbuch/spring-ai-agent-starter) —
  what an agent looks like in a production JVM service: a bounded, reactive
  tool-calling loop with retries, MCP mounting, and a unit-tested core.

**Observability & cost** *(decorator seams on the core)*
- [agent-meter](https://github.com/hhagenbuch/agent-meter) — OpenTelemetry-native
  token/cost attribution (per feature/session/prompt-version) with budgets that
  degrade before they deny. Add the dependency, get cost telemetry.
- [agent-blackbox](https://github.com/hhagenbuch/agent-blackbox) — the flight
  recorder: what the agent actually did, exportable for diffing and eval.

**Resilience**
- [castaway](https://github.com/hhagenbuch/castaway) — an agent runtime for
  disconnected/degraded environments: local-model failover, queued side-effects
  with revalidation, honesty under degradation.

**Delivery**
- [agent-operator](https://github.com/hhagenbuch/agent-operator) — a Kubernetes
  operator that treats a prompt like code: canary a new prompt version, gate the
  promotion on an eval Job, roll back on failure.

**Quality gates & contracts**
- [agent-evals](https://github.com/hhagenbuch/agent-evals) — JUnit for LLM agents:
  golden datasets, LLM-as-judge, exit-code CI gates (used across the platform).
- [mcp-pact](https://github.com/hhagenbuch/mcp-pact) — Pact-style contract testing
  for MCP servers, so a tool schema change can't silently break its consumers.

📫 heyward360@gmail.com
