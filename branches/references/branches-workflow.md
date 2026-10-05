# Branches execution workflow

This is the proposed Branches workflow implemented as instructions for a host agent. FreeLLMAPI supplies model inference and routing. The host supplies the coding environment and enforces this workflow.

## 1. Establish the run

Inspect repository instructions and current changes before preparing worktrees. Reuse requirements and budgets already supplied by the user. Resolve only missing information that prevents defining acceptance or safely using a paid route.

Record a compact run brief:

- Task, requested behavior, relevant constraints, exclusions, and acceptance criteria.
- Repository root, initial HEAD, source branch, staged/unstaged changes, and relevant untracked files.
- Shared baseline identifier; A/B worktree paths and collision-free branch names such as `branches/<run-id>/a` and `branches/<run-id>/b`.
- Actual A/B/debugger/fixer identities, selection evidence, context capacity, usable quota information, and free/paid classification.
- Existing transport/client, requested model or verified route, spending limit when applicable, test commands, and current stage.

Store non-secret run notes with the host's task artifacts, outside candidate source changes. They are execution artifacts, not extra files this skill package needs. Never store raw provider keys, credential-bearing URLs, or private environment dumps.

### Model selection

Filter availability before comparing capability. The global catalog and a model's advertised free offer do not establish the user's access. Exclude disabled, keyless, exhausted, non-code/non-text, and unresolved virtual entries. A `ready` status is an opportunity to try the model, not proof that a lazily checked key will succeed.

Resolve aliases and model families through available metadata. The same underlying model at two providers fills one identity. Do not claim independence for opaque aliases whose underlying identity cannot be established.

For eligible models, compare current evidence of coding/reasoning performance first, adequate context including output reserve as a prerequisite, observed reliability and quota headroom next. Prefer different families for A/B when capability is comparable. Use documented model information or available router capability metadata; do not invent benchmark scores or assume that `/v1/models` exposes every ranking field. With weak evidence, state the selection's uncertainty. No permanent model-name list belongs in the skill.

Assign distinct debugger and fixer models from remaining capable candidates when possible. With three models, C debugs both branches, A fixes A, and B fixes B. With two, B debugs A and A debugs B; the original builder fixes its own candidate. Keep diagnostic and implementation outputs separate even when a model fills several roles.

### Cost bounds and fallback

Free use requires verified free allowances on every route that could serve the request. Treat unknown pricing as unknown, not zero. Do not let a mixed fallback chain turn a free-only request into a paid call.

For paid use, the user must supply a limit with currency and scope; interpret an unqualified limit as the entire Branches run. Before each call, reserve a defensible maximum charge covering input, capped output, the permitted retries/fallbacks, and any other billable units. Count outstanding reservations when calls overlap. Update the ledger from available actual usage; keep uncertain charges reserved. Stop paid calls before the next reservation would exceed the limit.

Use only a transport whose routing and spending bounds can be verified. A model pin or a response header checked after execution does not prevent an unwanted charge. If the existing client cannot bound routes or charges, exclude the paid path and use an eligible free-only path. If none exists, report the limitation. This instruction package does not add gateway-side budget enforcement or modify routing profiles automatically.

### Served-model checks

Capture actual served identity through the existing MCP result or REST routing metadata, not through a model's self-description. Decode identifiers as required by the transport. Compare normalized underlying identities rather than provider names.

Maintain an author set per candidate containing every model whose build/fix patch was applied. Record diagnostic contributors separately. Reject an audit served by anyone in its target's author set. An audit with unknown identity is unverified and cannot satisfy the independent-review requirement. Do not apply build/fix output with unknown served identity: its authorship cannot be checked against the other candidate or a later reviewer.

Unexpected builder/fixer identity changes require reassessing the role map and independence before applying the output. If the fallback makes both builds share an author, discard the conflicting generation and request an eligible distinct builder. Preserve already accepted outputs; never relabel a fallback as the originally requested model. After one discovery refresh and one eligible retry for a failed stage, report persistent failure rather than loop indefinitely. Honor cooldown information instead of hammering exhausted routes.

## 2. Prepare identical baselines

Use a Git repository with an existing commit. A non-Git directory or unavailable worktrees is a prerequisite blocker; do not silently initialize a repository.

Capture relevant tracked changes against HEAD, including staged and unstaged content, plus task-relevant untracked files. Avoid copying secrets and unrelated generated files into model context. Preserve the original checkout and index exactly during construction.

Create an isolated baseline worktree from HEAD, apply the captured tracked delta there, and copy eligible untracked files. If helpful, create a local baseline snapshot commit only in that isolated worktree; do not stage or commit the user's source checkout. Create A and B from that same snapshot. Verify that their task-relevant starting contents match. If the baseline cannot be reproduced consistently, stop before model construction.

Record which files were omitted and why; if an omitted file is required for the task, resolve that prerequisite instead of proceeding on different starting code. Do not share credentials with model prompts. Use local environment configuration for test dependencies where appropriate.

Run the relevant checks on the baseline when feasible to distinguish pre-existing failures. Fix the acceptance criteria and tests used to compare both branches; candidates may add meaningful regressions, but cannot weaken the common tests to win. Preserve a record of known baseline failures.

Worktrees isolate files, not shared databases, services, or build ports. Run checks sequentially when their shared resources could interfere. Run model construction concurrently only when the existing client and execution environment support it; sequential independent construction is valid.

## 3. Build A and B

Give both builders identical task context and relevant baseline files, without the other builder's output. Use separate conversations/session identifiers where supported. Request manageable patches with enough context to apply them.

### Builder prompt

```text
You are the builder for this candidate. Implement the shared task against
the supplied baseline. Follow the repository constraints and acceptance
criteria. You have not been given the competing implementation.

Return the proposed file changes as an applicable patch or explicit file
edits, a short rationale, and suggested checks. If more repository context
is required, identify the exact files or observations you need.

The host applies edits and runs tools. Distinguish proposed checks from
observed results; do not claim a command ran unless tool evidence is supplied.
```

The host supplies requested context, checks served identity, applies the edits only in that builder's worktree, and executes checks. For existing tool-capable clients, use the actual tool schemas and return observed tool results to the same candidate conversation. Tool-call support does not itself execute a command.

Keep each candidate's diff relative to the shared baseline. Include new files and deletions. Record observed commands, exit status, and relevant failures. Reject malformed or unrelated patch content rather than applying it blindly.

## 4. Cross-audit

After both builds finish, send B's diff and evidence to A; send A's to B. Include shared acceptance criteria and relevant surrounding code. Each auditor works read-only with respect to its target.

### Auditor prompt

```text
Audit the supplied candidate against the shared requirements and baseline.
Find actionable correctness, regression, or relevant security defects.
Separate pre-existing behavior from defects introduced by this candidate.

For each finding give: severity, file and line/symbol, defect, consequence,
and reproduction/test evidence. When execution is unavailable, give a
concrete static reasoning trace and mark the finding unconfirmed.

Return no findings when none are supported. Do not invent test results or
rewrite the candidate during the audit. State coverage gaps explicitly.
```

Validate that the auditor is outside the target's author set before accepting the review. Assign stable finding IDs locally, distinguish confirmed defects from hypotheses, and carry open findings forward.

## 5. Debug, fix, and retest

Skip repair when the candidate meets acceptance and has no actionable defects. Otherwise run at most three repair cycles per candidate. One cycle comprises diagnosis, a scoped patch, host retesting, and review of the changed code. Role-specific context stays within the target candidate; do not combine competing implementations implicitly.

### Debugger prompt

```text
Diagnose the supplied candidate's failing checks and audit findings.
Return a minimal reproduction, the likely root cause, supporting evidence,
and a focused regression-check proposal. Separate confirmed observations
from hypotheses. Request missing logs or code precisely.

Do not modify the code. The host will run any proposed diagnostic commands
and return their results.
```

### Fixer prompt

```text
Fix the supplied confirmed defects in this candidate. Use the diagnosis
and current candidate code. Return a minimal patch and explain which
finding IDs it addresses. Add a regression check where it verifies the
failure meaningfully. Preserve the shared requirements and acceptance tests.

Do not broaden the task or claim proposed tests already passed. Request
missing context instead of guessing at unseen code.
```

The host verifies served identity, applies the patch, updates the author set, and reruns the affected checks plus required acceptance checks. An eligible auditor outside the updated author set reviews the changes. Do not count an author reviewing their own patch as independent verification. If reuse/fallback leaves no independent reviewer, retain the candidate as unverified.

Stop at the cycle limit, exhausted budget/capacity, or persistent transport failure. Keep the latest diffs, observations, open findings, and worktree locations. An infrastructure failure or unavailable dependency is a validation gap; do not label it a passing test.

## 6. Select and apply

A candidate is eligible to win only when it meets the agreed acceptance criteria, required checks have observed results, and the required independent review is valid. Pre-existing failures may be accepted only when that treatment was established in the shared brief; new required-test failures disqualify the candidate.

Compare eligible candidates in this order:

1. Acceptance-test evidence and absence of regressions.
2. Unresolved actionable audit findings, prioritizing greater severity.
3. Maintainability and fit with repository conventions.
4. Smaller task-related change size, excluding generated output.
5. Candidate A if still exactly tied.

Explain the decision using evidence, not the model's brand or self-reported confidence. Do not automatically blend patches or create a third candidate. If neither candidate qualifies, apply neither and report what prevents completion.

Compute the winning delta relative to the shared baseline so captured user edits are not applied twice. Re-inspect the source checkout for changes made since capture. Check whether the entire winning delta can apply before modifying files, including new-file collisions and deletions. Use a check-only operation such as `git apply --check` for the complete patch, supplemented by file checks as needed.

On a clean application, apply only that delta without staging the user's files or switching their source branch. Preserve unrelated content, file modes where applicable, and the existing index. Rerun the agreed checks in the source checkout. If new source changes conflict, retain the winning worktree and report the conflicting files; do not reset, force-apply, or overwrite user work.

If verification fails after application, retain the evidence and report that final validation failed. If rollback is needed, reverse only the skill's delta after checking that doing so preserves subsequent user edits; otherwise leave the recoverable changes and explain the state. Do not report a verified winner while this remains unresolved.

Keep candidate worktrees and patches on failure. After success, preserve recoverable candidate snapshots before any task-worktree cleanup. Never delete a user's existing checkout, branch, or worktree merely to tidy this run. No publishing, pushing, deployment, or global configuration change is implied.

## 7. Report the outcome

Include applied behavior and why the candidate won, actual requested/served identities and support-role reuse, meaningful tests with observed outcomes, unresolved findings or validation gaps, repair cycles used, and recoverable artifact paths. Report token/cost information only when available and mark uncertainty. Include remaining budget when paid accounting supports it.

## Behavioral review scenarios

These are review cases for the instruction package, not claims of live execution.

| Scenario | Expected behavior |
| --- | --- |
| Four eligible identities | A/B build and cross-audit; C diagnoses; D fixes; A/B re-audit the opposite candidate. |
| Three identities | C diagnoses both; A fixes A and B fixes B; cross-audits remain independent. |
| Two identities | Opposite builder diagnoses; original builder fixes; disclose reduced independence. |
| Fewer than two | Report insufficient distinct eligible models; do not simulate two providers with one host model. |
| Frontier access unverified | Choose the strongest supported available pair and state uncertainty. |
| Catalog includes virtual models | Remove them from candidate identity counts. |
| Audit falls back to its target's author | Discard the audit and retry with an eligible distinct identity; persistent failure blocks qualification. |
| Served identity is missing | Treat independence as unverified; do not declare an audited winner. |
| Free quota exhausted | Refresh availability once, respect cooldowns, and use only another eligible route; never silently spend. |
| Paid key but no spending limit | Exclude paid calls and use verified free routes if available. |
| Paid route or retry cost cannot be bounded | Exclude that path even when a spending limit exists. |
| User purchased the live catalog | Do not infer a paid inference entitlement or quota increase. |
| Both candidates fail after three cycles | Apply neither; preserve artifacts and report unresolved failures. |
| One candidate passes | Apply it only after valid independent review and a clean application check. |
| Source has pre-existing edits | Include relevant edits in both baselines; apply only the winner's incremental delta and preserve the index. |
| User edits source during construction | Recheck applicability; preserve candidates and report conflicts without overwriting. |
| Candidate claims tests passed without logs | Host must execute and observe checks; model claims do not count. |
