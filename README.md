# X-SuperPlay Agent Control Plane

Evidence-driven orchestration for AI-native software development.

This project coordinates planning, task dispatch, isolated implementation workers, evidence collection, and exact-head acceptance. It is intentionally **not** a model/API gateway.

## Product objective

Build a development control plane in which:

- **ChatGPT Web** performs high-level planning, architecture, PRD refinement, and review;
- **Codex** acts primarily as controller, task decomposer, dispatcher, and acceptance reviewer;
- **Claude Code** is the primary implementation worker;
- **Google Antigravity CLI** can act as an independent secondary worker through its official headless/CLI path;
- additional CLI agents can be added behind the same worker contract;
- each implementation task runs in an isolated Git worktree;
- completion is based on structured evidence, not worker self-report;
- native Codex, Claude Code, and Antigravity remain independently usable if this project fails.

## Target architecture

```text
                         PLANNING PLANE
                       ChatGPT Web
                  architecture / PRD / review
                              |
                         plan artifact
                              v
                  +-------------------------+
                  | Agent Control Plane     |
                  |                         |
                  | Codex Controller        |
                  | task / dispatch / state |
                  +-----------+-------------+
                              |
                +-------------+-------------+
                |                           |
                v                           v
       Claude Code Worker          Antigravity Worker
          (`claude -p`)              (official `agy`)
                |                           |
                +-------------+-------------+
                              |
                              v
                    isolated Git worktrees
                              |
                    diff / tests / commit
                              |
                              v
                       Evidence Bundle
                              |
                              v
                        Codex Reviewer
                         PASS / REJECT
```

A separate model-access gateway may be used by a worker when explicitly configured, but it is **optional**. This control plane must support native/direct worker launch and must not require a proxy to exist.

## Responsibilities

This repository owns:

- ChatGPT Web planning-artifact handoff;
- Codex controller/reviewer workflow;
- Claude Code worker delegation;
- Antigravity official-CLI worker delegation;
- generic CLI worker contracts;
- one-task-one-worktree isolation;
- Task Contract, Evidence Bundle, and Acceptance Result schemas;
- task dependency and parallel-scope control;
- task/review drift prevention;
- bounded worker lifecycle, progress, cancellation, and result collection;
- exact-head acceptance and post-merge verification;
- auditability of plan revision, base SHA, worker identity, and reviewed head SHA.

## Non-goals

This project must not become:

- a general-purpose model/API gateway;
- a credential aggregation service;
- a mechanism for evading provider quotas or account controls;
- a replacement for Git;
- a monolithic runtime whose failure disables native development tools;
- a system where a worker saying "done" is accepted without independent evidence.

## Core workflow

```text
User intent
   |
   v
ChatGPT Web plan
   |
   v
.ai-control/current-plan.md + acceptance.yaml
   |
   v
Codex controller
   |
   +--> TASK-001 -> worktree/TASK-001 -> Claude Code
   +--> TASK-002 -> worktree/TASK-002 -> Antigravity
   |
   v
Evidence Bundles
   |
   v
Codex exact-head review
   |
   +--> PASS -> merge/promote -> post-merge retest
   +--> REJECT -> bounded remediation against the same objective
```

## Planned repository structure

```text
X-SuperPlay-Agent-Control-Plane/
├── README.md
├── AGENTS.md
├── docs/
│   ├── architecture.md
│   ├── reference-projects.md
│   ├── roadmap.md
│   ├── contracts.md
│   └── provenance.md
├── schemas/
│   ├── task-contract.schema.json
│   ├── evidence-bundle.schema.json
│   └── acceptance-result.schema.json
├── controller/
│   └── codex/
├── workers/
│   ├── claude-code/
│   └── antigravity/
├── bridges/
│   └── chatgpt-web/
├── worktrees/
├── acceptance/
└── tests/
```

Runtime worktrees, credentials, OAuth/session state, logs containing secrets, and local agent profiles must not be committed.

## Reference projects

The initial architecture is informed by these projects. They are **references, not automatic dependencies**:

- [`Qihao-Duan/claude-code-agent-for-codex`](https://github.com/Qihao-Duan/claude-code-agent-for-codex) — Codex delegates to the full `claude -p` agent loop over MCP; useful for Claude worker lifecycle, async jobs, progress, cancellation, runtime profiles, and permission tiers.
- [`thomaswitt/mcp-agents`](https://github.com/thomaswitt/mcp-agents) — shared MCP-style delegation surface for Codex, Claude Code, and Antigravity; useful for provider adapters, long-running jobs, cancellation, result paging, and durable controller-side state.
- [`IlleJiViN/codex-antigravity-subagent`](https://github.com/IlleJiViN/codex-antigravity-subagent) — Codex delegates bounded work to locally authenticated official `agy`; useful for safe Antigravity delegation and plan/edit modes without bypassing native auth or permissions.
- [`naplesblue/codexbridge`](https://github.com/naplesblue/codexbridge) — controlled ChatGPT Web ↔ local repository context and plan-handoff artifacts; useful for bounded repository exposure, `.ai-bridge/current-plan.md`-style handoff, safe local execution separation, and compact evidence flow.
- [`Dalomeve/codex-chatgpt-bridge`](https://github.com/Dalomeve/codex-chatgpt-bridge) — additional reference for ChatGPT/Codex local bridge patterns.

See [`docs/reference-projects.md`](docs/reference-projects.md) for the adoption matrix and what should or should not be copied from each project.

## Development phases

```text
P0  Architecture, governance, reference inventory
P1  Task / Evidence / Acceptance schemas
P2  Worktree manager and scope validation
P3  Claude Code worker adapter
P4  Codex controller / reviewer state machine
P5  Parallel dependency and drift controls
P6  Antigravity official-CLI worker adapter
P7  ChatGPT Web planning bridge
P8  Cross-plane failure isolation, recovery, audit, operations
P9  Exact-head end-to-end acceptance and post-merge retest
```

The authoritative task breakdown and gates live in [`docs/roadmap.md`](docs/roadmap.md).

## Completion gate

This project is not complete merely because Codex can call a Claude model or execute `claude` once.

```text
AGENT_CONTROL_COMPLETE =
    planning_artifact_protocol_defined
AND task_contract_schema_pass
AND evidence_bundle_schema_pass
AND acceptance_contract_schema_pass
AND codex_controller_dispatch_pass
AND claude_code_worker_pass
AND claude_worker_isolation_pass
AND worktree_per_task_pass
AND parallel_scope_isolation_pass
AND task_revision_drift_protection_pass
AND evidence_not_transcript_is_completion_basis
AND exact_head_review_pass
AND reject_remediation_scope_preserved
AND post_merge_retest_pass
AND antigravity_official_cli_worker_pass
AND chatgpt_web_handoff_pass
AND model_gateway_optional
AND native_codex_survival_independent
AND native_claude_survival_independent
AND official_agy_survival_independent
```

Completion claims without exact artifact identity, test evidence, scope evidence, and post-merge verification are not completion evidence.

## Migration provenance

The initial design was extracted from architecture work previously recorded in `yandexuanxuan/codex-proxy-suite` PR #25, merged as commit `70eba44917fcfd9000491dd6ebbd52b54abb0140`. This repository is now the intended authoritative home for Agent Control Plane work; the source repository should remain focused on model access only.
