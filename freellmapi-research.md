# FreeLLMAPI research and suitability for Branches

Research date: **October 5, 2026** (Asia/Manila).

This report reviews the project's public documentation and selected source code. It does not establish live provider access, independently benchmark models, or verify inference on a running installation. Documentation and catalog data can change.

## What FreeLLMAPI provides

FreeLLMAPI is an MIT-licensed, self-hosted gateway combining user-configured provider access behind a unified API. The repository lists Google, Groq, Cerebras, Mistral, OpenRouter, NVIDIA, and other integrations, plus custom OpenAI-compatible endpoints. Provider access comes from the user's configured credentials and upstream allowances. The gateway does not create a universal entitlement to frontier models. [Project repository](https://github.com/tashfeenahmed/freellmapi)

### Catalog claims and conflicting counts

The website displayed **855 models across 51 providers** during this review. The repository README displayed **34 providers, 635 free endpoints, and 474 model families**. Both advertised **7.4 billion tokens per month**. These are project claims, not measured capacity available to a particular user. [Website](https://freellmapi.co/), [README](https://github.com/tashfeenahmed/freellmapi/blob/main/README.md)

The documents count families, endpoints, and models differently and do not reconcile the totals. Catalog timing and differing definitions could explain part of the discrepancy, but that is an inference. Branches should use the running installation's usable-model inventory rather than either headline.

The README describes free catalog entries arriving after a 30-day delay, with premium access receiving the live feed. The website advertised $19 annually or $49 for lifetime catalog access. Purchasing catalog freshness does not establish increased upstream inference allowance or paid-model entitlement. Recheck prices and offers before purchase. [README](https://github.com/tashfeenahmed/freellmapi/blob/main/README.md), [Website pricing](https://freellmapi.co/#pricing)

## Architecture, keys, and quotas

The documented stack uses an Express proxy, provider adapters, a React dashboard, and SQLite storage. Provider credentials use AES-256-GCM encryption at rest and are decrypted for upstream calls. A local unified bearer token authenticates clients. Local key storage does not mean local inference: requests reach enabled upstream providers. [Architecture guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/architecture/00-high-level-index.md), [Client guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/clients/01-agent-clients.md)

The architecture guide describes capability, speed, reliability, and quota-aware routing, with rate tracking for requests/tokens per minute/day. Provider errors can cause cooldown and failover. Free allowances are ceilings, not reserved capacity. Access, latency, and quotas may change, so sustained frontier access cannot be assumed. The project describes itself as single-user, with no multi-tenant authentication or SLA. [Architecture guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/architecture/00-high-level-index.md)

## Windows setup

The project's installation guide identifies its Windows desktop `.exe` as the easiest path. Download it from the repository's Releases, launch the local router, add provider keys through the dashboard, and obtain the unified key from the tray or Keys page. Desktop state is documented under `%APPDATA%\FreeLLMAPI\`. [Installation guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/install/01-install.md)

Docker Compose and Node.js 20+ development are alternatives. The guide provides PowerShell instructions. Development uses a Vite dashboard on port 5173; the built server/dashboard uses port 3001. Building the Electron desktop app on Windows requires Python and Visual Studio's C++ build tools. Preserve the encryption key and database when upgrading; replacing the encryption key alone does not migrate encrypted credentials. [Installation guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/install/01-install.md)

These are setup options for the user; creating the Branches package does not install or configure the gateway.

## API discovery and routing

The documented OpenAI-compatible base URL is `http://localhost:3001/v1`. Relevant endpoints include `/chat/completions`, `/responses`, and `/models`. Model listing includes unavailable catalog entries; `?execution_status=ready` narrows real rows to currently routable models, including keys not yet probed. Virtual `auto`, `fusion`, and profile entries can remain in filtered results. [API reference](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/api/01-rest-api.md)

Routing options include `auto:smart`, `auto:fast`, `auto:reliable`, `auto:balanced`, and named profiles. Routing preference is not an independent coding benchmark. `X-Routed-Via` and fallback headers expose actual service identity and routing changes. [API reference](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/api/01-rest-api.md)

Source inspection shows a preferred explicit model moved to the front of a routing chain. Therefore, a requested identity alone should not be treated as a hard routing restriction. The installed version and dispatch path must be checked. Named-chain documentation also distinguishes curated profiles from broad auto routing. These details matter for both independence and spending control. [Router source](https://github.com/tashfeenahmed/freellmapi/blob/main/server/src/services/router.ts), [Named fallback chains](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/fallback/01-named-chains.md)

## Client and MCP integration

The project documents compatible coding clients and setup generators with dry-run support. Its client guide assigns Codex CLI to the Responses surface and Claude Code to Anthropic Messages. Client-specific base URLs and protocol support should be checked against the installed versions. These are FreeLLMAPI's integration instructions, not proof that every client/model combination has been tested. [Client guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/clients/01-agent-clients.md)

The guide describes a Streamable HTTP MCP surface at `/mcp`, disabled on fresh installs until enabled. It can expose inference and gateway introspection. `ask_freellmapi` returns answer and execution metadata, including served model and usage. A connected host should inspect actual tool schemas before use. Neither an inference response nor support for tool-call messages supplies filesystem operations or executes tests by itself. [Client guide](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/clients/01-agent-clients.md)

## Fusion and the boundary with Branches

The documented `fusion` virtual model sends a prompt to a model panel and uses a judge to synthesize the drafts. Each sub-call consumes its normal quota. This is useful evidence that the project supports multi-model inference, but it does not document the independent Git builds, reciprocal code audits, diagnostic role, patching role, or winner application requested here. [API reference](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/api/01-rest-api.md)

**Design inference:** Branches can use FreeLLMAPI as its model-access layer. A host agent must supply worktrees, context handoffs, patch application, test execution, author tracking, and comparison. Fusion's synthesized answer cannot replace two inspectable candidates with attributable authorship.

## Proposed Branches design

The following describes this skill's proposed behavior, not an existing FreeLLMAPI feature:

| Role | Responsibility |
| --- | --- |
| A | Build candidate A; audit candidate B. |
| B | Build candidate B; audit candidate A. |
| Debugger | Reproduce failures and diagnose causes without patching. |
| Fixer | Implement scoped fixes from confirmed diagnosis. |
| Host | Manage tools, isolation, costs, served identities, checks, and final application. |

Choose the strongest available distinct pair dynamically. Prefer four models, but reuse support roles when only two or three are eligible. Track actual authors to prevent fallback self-review. Default to verified free routes; include configured paid access only within a supplied and enforceable spending limit. Preserve user edits through identical starting snapshots and apply only the winning incremental delta.

The [execution workflow](branches-workflow.md) supplies role prompts, three-cycle repair limits, selection criteria, and failure handling. The package is ready for installation as a skill folder; it supplies no new runtime, connector, or paid access. Its structure follows the [official OpenAI skill guidance](https://developers.openai.com/plugins/build/skills).

## Practical limitations and recommendation

The main constraint is reliable access and attribution, not headline catalog size. Weak availability, opaque aliases, unknown pricing, or missing tools can prevent Branches from completing safely. Project documentation describes personal use and provider-specific restrictions; this review does not independently assess each provider's terms. Recheck the intended provider's current requirements for an actual deployment. [Architecture limitations](https://github.com/tashfeenahmed/freellmapi/blob/main/docs/en/architecture/00-high-level-index.md)

**Recommendation:** use FreeLLMAPI for a private experimental Branches workflow when the host can verify two distinct usable coding models, permitted fallback routes, and observed test results. Treat the research as a dated integration assessment. Live availability, coding quality, and budget behavior still need validation on the user's actual configuration.
