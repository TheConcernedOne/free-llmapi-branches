---
name: branches
description: Coordinate two independent coding solutions through the user's FreeLLMAPI models, cross-audit them, debug and fix defects, and apply the tested winner. Use for the Branches coding workflow; ordinary code edits and research alone do not require it.
---

# Branches

Use the user's best available coding models to build two solutions from the same starting code. Model A builds candidate A and audits B; model B builds candidate B and audits A. Assign other capable models to debugging and fixing when available. The host agent manages files, worktrees, tools, tests, and the final application.

This package contains instructions and references. It does not install FreeLLMAPI, provide an API helper, or implement an orchestration service.

## Read what is needed

- Read [the workflow](references/branches-workflow.md) before executing Branches. It contains selection rules, role prompts, handoffs, repair limits, and checkout handling.
- Read [the research report](references/freellmapi-research.md) for architecture, documented interfaces, setup options, and limitations. Its October 5, 2026 availability information must be rechecked when needed.

## Prerequisites

Require a Git repository with a commit, worktree support, host file/terminal tools, and an existing FreeLLMAPI connection or compatible client that can request specific models and expose actual served-model metadata. A text-only inference tool is sufficient when the host can apply returned patches and run tests. Installing this skill does not supply that connection.

Inspect the available tools and their current schemas. Use existing credentials without printing or embedding them in artifacts. If a required capability is missing, report the specific prerequisite and retain any work already done. Do not invent tool names, assume a host subagent uses a different provider model, or silently substitute the host's own reasoning for A/B.

## Select and account for models

1. Discover real, currently usable text/code models through the connection's model-listing tool or authenticated `GET /v1/models?execution_status=ready`. Remove `auto`, `auto:*`, `fusion`, and unresolved discovery aliases. Deduplicate the same model served by multiple providers.
2. Choose the strongest two distinct coding models that fit the task and policy. Prefer different families; use current coding evidence, adequate context, reliability, and quota headroom. Explain uncertainty and call them the best available candidates when frontier access is unverified.
3. Prefer distinct debugger and fixer models. With three models, use the third as debugger and each candidate's builder as its fixer. With two, use the opposite builder as debugger and the original builder as fixer. Disclose support-role reuse; fewer than two eligible identities blocks execution.
4. Default to verified free allowances. Configured paid inference is eligible only with a user-supplied spending limit and enforceable cost bounds. Paid catalog access does not establish paid inference access. Check the complete possible fallback route before calling; if costs or allowed routes cannot be bounded, exclude that path.
5. Request explicit model identities and record the actual provider/model, fallbacks, usage, and available cost for every accepted output. Treat model identity as independent of provider identity. Reject an audit served by a model that authored or patched its target; unknown identity cannot establish an independent audit.

## Execute

1. Establish shared requirements, acceptance criteria, test commands, and an identical baseline including relevant existing user changes. Build A/B in separate worktrees without changing the source checkout's branch or index.
2. Give both builders the same baseline context. Keep their implementations separate until construction is complete. Have the host apply only candidate-specific patches and run the same acceptance checks.
3. Cross-audit: A reviews B and B reviews A, with read-only access to the target diff and test evidence. Require concrete findings with location, consequence, and reproduction or static evidence.
4. Debug and fix each candidate independently. Allow at most three debug/fix/retest cycles per candidate. Re-audit changed code with a model outside that candidate's author set. Stop sooner when budget or usable capacity is exhausted.
5. Select among candidates meeting acceptance criteria: compare test evidence, unresolved audit findings, maintainability, then change size. Use A for a final exact tie. Neither passing means no winner.
6. Apply only the winning delta relative to the shared baseline to the user's working checkout. Check applicability first, preserve unrelated edits and index state, and rerun required checks there. On conflict or failed verification, retain recoverable candidate artifacts and report the blocker without claiming completion.

## Report

Summarize the actual model assignments and any reuse/fallback, why the winner was selected, applied changes, test commands/results, unresolved findings, artifact locations, and available usage/cost. Label unavailable cost or identity data as unknown. Do not claim tests were executed based on model text alone.
