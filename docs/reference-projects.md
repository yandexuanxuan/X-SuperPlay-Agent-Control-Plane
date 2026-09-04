# Reference Projects and Adoption Matrix

Status: **authoritative benchmark/reference inventory for Agent Control development**  
Last reviewed: 2026-09-05

The projects below are design references for the two Agent Control requirements:

1. **Codex controls/reviews while Claude Code and other agents execute**;
2. **ChatGPT Web performs high-level planning/review and hands bounded artifacts to local execution**.

They are not automatically vendored dependencies. The goal is to systematically harvest proven ideas that mature projects already implement: job lifecycle, native event fidelity, permissions, session identity, recursion guards, plan handoff, tunnel/auth safety, usage display, and protocol capability negotiation.

## Adoption rule

```text
reference != dependency
```

For every proposed import or copied pattern:

1. identify the exact capability needed;
2. verify current license, provenance, maintenance state, release cadence, and security boundary;
3. prefer interface/protocol adaptation over vendoring;
4. pin exact upstream identity if code is imported;
5. keep this project's Task/Evidence/Acceptance contracts authoritative;
6. keep one-task-one-worktree and exact-head acceptance authoritative;
7. preserve native Codex / Claude Code / `agy` independence;
8. add deterministic acceptance coverage for the adopted behavior.

---

# Capability C — Codex controller/reviewer -> Claude Code / multi-agent execution

## C1. Qihao-Duan/claude-code-agent-for-codex

Repository:

- https://github.com/Qihao-Duan/claude-code-agent-for-codex

### Why it matters

This project exposes **Claude Code's full agent loop** to Codex over MCP by wrapping `claude -p`, rather than mirroring individual Read/Edit/Bash primitives.

Core direction:

```text
Codex -> MCP -> bridge -> claude -p -> Claude Code agent loop
```

### Patterns to evaluate/adopt

- real Claude Code agent-loop delegation;
- sync path only for small bounded jobs;
- async `start -> status -> result` path for real coding work;
- durable job ids and persisted job state;
- progress/heartbeat reporting;
- cancellation;
- structured phase/failure classification;
- explicit working directory;
- runtime profiles controlling inherited local state;
- permission tiers and deny lists;
- bounded stdout/stderr/log paths;
- budget/timeout parameters.

### What not to inherit blindly

- its MCP schema does not replace our Task Contract/Evidence Bundle;
- worker exit success is not acceptance;
- worktree ownership, base SHA, plan revision, changed-path scope, tests, and exact-head review remain project-owned gates.

Target area:

```text
workers/claude-code/
```

Priority: **HIGH**.

---

## C2. thomaswitt/mcp-agents

Repository:

- https://github.com/thomaswitt/mcp-agents

### Why it matters

`mcp-agents` demonstrates one wrapper-owned integration surface over heterogeneous CLI agents, including Codex, Claude Code, and an Antigravity-backed provider.

### Patterns to evaluate/adopt

- normalized worker/provider adapter interface;
- long-running background job lifecycle;
- `start -> status -> result -> cancel` semantics;
- bounded result paging;
- liveness separate from final output;
- durable/resumable controller-side state where useful;
- wrapper-owned stable contract over changing underlying CLI protocols;
- no automatic fallback between semantically different agent providers;
- adapting newer Codex App Server behavior behind a stable outer boundary.

### What not to inherit blindly

- this project is not a generic model gateway;
- we do not need every provider initially;
- upstream state semantics do not replace Git/worktree/exact-head governance.

Target areas:

```text
worker adapter interface
controller job lifecycle
future generic CLI workers
```

Priority: **HIGH**.

---

## C3. BytePioneer-AI/codex-host

Repository:

- https://github.com/BytePioneer-AI/codex-host

### Placement decision

**This project belongs in `X-SuperPlay-Agent-Control-Plane`, not in the Model Access Gateway.**

Its core concern is hosting multiple **Agent Harnesses** (Pi, Claude Code and additional harnesses) inside the Codex Desktop experience while preserving agent-native capabilities. It is about agent execution/session/UI integration rather than provider API routing.

### Why it matters

CodexHost is especially valuable because it tackles a problem our initial design under-specified: **integration fidelity**. A generic protocol can make multiple agents callable while losing native semantics such as approvals, diffs, questions, tool state, thinking/model status, session restore, fork, and context compaction.

CodexHost instead integrates each harness through its native interface and projects native events into Codex Desktop.

### Patterns to evaluate/adopt

- **native-event fidelity** rather than lowest-common-denominator normalization;
- per-harness adapter that preserves its own capabilities;
- explicit capability matrix for each worker/harness;
- streamed tool state, reliable edit diff, approvals, questions, cancellation;
- model/thinking state surfaced independently from final output;
- session resume/thread identity/fork semantics;
- context compaction state;
- usage/cost display per harness where available;
- cross-platform launcher/process discovery;
- GUI-start PATH differences and executable discovery diagnostics;
- graceful shutdown with bounded forced cleanup;
- startup tracing and lifecycle phase visibility;
- host UI can expose multiple independent agent sessions without pretending they are the same harness;
- credentials remain owned by the local harness rather than centralized into the host.

### High-value backlog candidates inspired by CodexHost

```text
worker_capability_descriptor
native_event_projection
approval_event_contract
question_blocking_state
session_resume_identity
worker_fork_semantics
context_compaction_event
usage_and_cost_metadata
startup_phase_trace
cross_platform_executable_discovery
```

### Important architecture lesson

A universal Worker Contract should normalize **lifecycle and evidence**, but it should not erase harness-specific capabilities.

Recommended model:

```text
common lifecycle:
  prepare/start/status/cancel/collect

plus capability-specific events:
  approval
  question
  diff
  tool_state
  usage
  thinking
  session_resume
  fork
```

### What not to inherit blindly

- we do not need to patch/augment Codex Desktop UI in the first implementation;
- CDP/Electron injection is a separate risk/update surface and should remain optional;
- our authoritative completion basis is Evidence Bundle + exact Git identity, not a desktop session UI;
- worker adapters must remain usable headlessly.

Target areas:

```text
workers/* capability model
controller event model
observability/session UX
future desktop integration
```

Priority: **HIGH**.

---

## C4. Dunqing/claude-codex-bridge

Repository:

- https://github.com/Dunqing/claude-codex-bridge

### Why it matters

This project implements a bidirectional MCP bridge between Claude Code and Codex CLI and provides setup automation, skills/agents, retry behavior, and an anti-recursion guard.

### Patterns to evaluate/adopt

- bidirectional delegation as an explicit topology;
- simple one-command registration/bootstrap;
- task-specific bridge tools (review, explain, plan, implement) rather than one untyped generic call;
- transient failure retry policy;
- recursion/delegation-depth guard to prevent Claude -> Codex -> Claude loops;
- teammate/subagent pattern for parallel second opinions;
- environment-scoped bridge depth metadata;
- separate setup for one-way vs two-way integration.

### High-value backlog candidates

```text
delegation_depth_guard
cyclic_delegation_detected
worker_retry_policy
one_way_vs_bidirectional_policy
task_intent_enum
```

### What not to inherit blindly

- our desired default is **Codex controller -> Claude worker**, not symmetric uncontrolled delegation;
- automatic retries must never replay an implementation turn whose outcome is unknown;
- review/implementation roles remain governed by Task Contract and acceptance policy.

Priority: **HIGH**.

---

## C5. agentclientprotocol/agent-client-protocol

Repository:

- https://github.com/agentclientprotocol/agent-client-protocol

### Why it matters

ACP standardizes editor/client <-> coding-agent communication. Even if this project does not make ACP its sole transport, ACP is a useful benchmark for a durable agent capability/event contract.

### Patterns to evaluate/adopt

- capability negotiation between client and agent;
- standardized permission/approval concepts;
- structured tool and edit events;
- agent/client separation;
- explicit protocol versioning/evolution;
- ecosystem registry/adapter approach;
- language SDK separation from core schema.

### Architecture lesson

Use ACP as a **comparison standard** for what a generic worker surface should express, while retaining project-specific Task/Evidence/Acceptance semantics.

Priority: **HIGH, PROTOCOL REFERENCE**.

---

## C6. agentclientprotocol/codex-acp

Repository:

- https://github.com/agentclientprotocol/codex-acp

### Why it matters

This is a current ACP adapter around Codex App Server and demonstrates detailed projection of Codex behavior into a generic agent protocol.

### Patterns to evaluate/adopt

- App Server hidden behind an adapter-owned contract;
- auth method negotiation;
- model/reasoning/sandbox/approval configuration;
- shell, file-change, permission, MCP, terminal, reasoning, plan, usage and review events;
- subagent launches represented as structured tool events with thread identity metadata;
- client-supplied MCP servers;
- version-gated regeneration/testing against upstream Codex changes;
- cross-platform standalone binary packaging.

### High-value backlog candidates

```text
worker_protocol_version
capability_negotiation
subagent_parent_child_identity
permission_request_event
review_event
usage_event
adapter_upstream_version_gate
```

### What not to inherit blindly

- ACP is not the Evidence Bundle;
- client protocol events do not replace independent Git/test acceptance;
- do not expose raw unstable App Server frames as our project contract.

Priority: **HIGH**.

---

## C7. IlleJiViN/codex-antigravity-subagent

Repository:

- https://github.com/IlleJiViN/codex-antigravity-subagent

### Why it matters

This project lets Codex use the locally authenticated official Antigravity CLI (`agy`) as an external agent without forcing the subscription through a generic model proxy.

### Patterns to evaluate/adopt

- `agy` availability/version check before dispatch;
- official CLI/headless process boundary;
- locally owned authentication;
- plan/read-only default where appropriate;
- explicit edit-capable mode;
- no exposed dangerous permission-bypass flag;
- bounded timeout/output;
- Codex remains responsible for verification/final acceptance.

Target area:

```text
workers/antigravity/
```

Priority: **HIGH**.

---

# Capability D — ChatGPT Web planning/review -> bounded local handoff

## D1. naplesblue/codexbridge

Repository:

- https://github.com/naplesblue/codexbridge

### Why it matters

CodexBridge connects ChatGPT Web/Developer Mode to bounded local repository context through MCP and supports artifact-based handoff such as `.ai-bridge/current-plan.md`.

### Patterns to evaluate/adopt

- bounded repository opening/context selection;
- read-mostly/minimal tool profiles;
- workspace-scoped writes;
- blocked secret/cache/build paths;
- plan artifact instead of copied full transcript;
- remote planning separated from user-started local execution;
- content hash/revision detection to prevent duplicate execution;
- compact status/diff/evidence output;
- operation journal/audit concepts;
- stable-vs-quick tunnel diagnostics;
- context-bundle fallback when the selected ChatGPT surface cannot call tools;
- explicit statement that the bridge does not bypass product limits.

### Target adaptation

Use `.ai-control/` rather than `.ai-bridge/`, and route execution through the Codex controller rather than directly to an unrestricted worker.

Priority: **HIGH**.

---

## D2. Dalomeve/codex-chatgpt-bridge

Repository:

- https://github.com/Dalomeve/codex-chatgpt-bridge

### Why it matters

This project explicitly models ChatGPT Web as researcher/planner/reviewer and local Codex as executor, with restricted grants as the safe default.

### Patterns to evaluate/adopt

- directory grant model (`read`, `write`, `execute` capabilities);
- sensitive-path blocking;
- restricted vs full-delegate modes;
- separate tool profiles for planning/delegation;
- tunnel/app health diagnostics;
- explicit distinction between host-side safety block and local bridge failure;
- easy rollback from broad mode to restricted mode.

### High-value backlog candidates

```text
workspace_grant_model
bridge_tool_profile
sensitive_path_policy
planning_only_mode
connector_health_status
host_block_vs_bridge_failure_classification
```

Priority: **HIGH**.

---

## D3. howlabs/bifrost

Repository:

- https://github.com/howlabs/bifrost

### Why it matters

Bifrost is small, but its architecture is almost exactly the minimal handoff path:

```text
ChatGPT -> MCP/tunnel -> saved plan artifact -> local Claude/Codex reads and executes
```

### Patterns to evaluate/adopt

- plan schema/required sections before saving;
- dedicated plan storage rather than free-form shell execution;
- allowed-directory list for repository reads;
- minimal tool surface (`save_plan`, `get_plan`, `list_dir`, `read_file`);
- separation between planning channel and implementation agent.

### Architecture lesson

The first ChatGPT bridge can be **much smaller** than a full remote coding environment. A plan-only MVP reduces security and failure surface.

Priority: **MEDIUM-HIGH, MVP REFERENCE**.

---

## D4. openai/openai-apps-sdk-examples

Repository:

- https://github.com/openai/openai-apps-sdk-examples

### Why it matters

This is an official reference for Apps SDK + MCP integration used by ChatGPT. It should be consulted for the ChatGPT-facing boundary instead of inferring host behavior solely from community projects.

### Patterns to evaluate/adopt

- tool list/call contracts with JSON Schema;
- structured tool content;
- tool annotations such as read-only hints;
- Streamable HTTP/SSE style server boundaries where relevant;
- Apps SDK UI resource/widget metadata if a status/approval UI is later added;
- local-server-to-ChatGPT development workflow;
- tunnel/DNS-rebinding considerations documented by official examples.

### High-value backlog candidates

```text
chatgpt_tool_annotations
read_only_tool_hints
structured_tool_result
chatgpt_widget_status_card
mcp_transport_contract_tests
```

### What not to inherit blindly

- UI widgets are optional and should not precede the plan/evidence protocol;
- the bridge must remain useful without a custom UI.

Priority: **HIGH, OFFICIAL INTEGRATION REFERENCE**.

---

## D5. DeepCogNeural/codex-gpt-bridge

Repository:

- https://github.com/DeepCogNeural/codex-gpt-bridge

### Why it matters

This project focuses on a Streamable HTTP MCP bridge from ChatGPT/GPTs to local Codex and documents operational details around loopback binding, tunnel exposure, bearer tokens, allowed hosts, DNS-rebinding protection, and stronger OAuth requirements for durable use.

### Patterns to evaluate/adopt

- loopback-first binding;
- explicit allowed workspace roots;
- bearer-token development mode;
- tunnel host allowlist / DNS-rebinding protection;
- stronger OAuth/PKCE boundary for durable protected deployments;
- bridge status and job status tools;
- explicit distinction between smoke-test no-auth and durable authenticated operation.

### High-value backlog candidates

```text
loopback_default
allowed_host_policy
tunnel_origin_validation
bridge_auth_mode
production_oauth_boundary
secret_free_status
```

Priority: **MEDIUM-HIGH, SECURITY/OPERATIONS REFERENCE**.

---

# Cross-capability patterns we should proactively evaluate

The references expose important design areas that the initial roadmap could otherwise miss.

## Worker capability descriptor

A worker should declare supported optional features instead of the controller assuming they exist:

```yaml
capabilities:
  edits: true
  tools: true
  approvals: true
  questions: true
  diff_events: true
  reasoning_events: true
  usage: true
  session_resume: true
  fork: false
  context_compaction: true
```

## Common lifecycle + native extension events

Normalize only the lifecycle:

```text
prepare
start
status
cancel
collect
```

Preserve richer native events separately:

```text
tool_state
approval_request
question
edit_diff
usage
reasoning_status
session_identity
fork
compaction
```

This avoids degrading every harness to the weakest common denominator.

## Delegation graph guard

Every run should record parent controller/task/worker identity and delegation depth. Cyclic chains must fail deterministically.

```text
Codex -> Claude -> Codex -> Claude ...
```

must not be possible accidentally.

## Unknown-outcome protection

If a worker/bridge dies after dispatch, do not auto-replay an implementation prompt until repository/process evidence establishes whether it ran.

## Planning bridge trust profile

The ChatGPT bridge should support progressively stronger profiles:

```text
plan-readonly
plan-write-artifacts
review-evidence
(optional later) bounded control action
```

Source-code writes and unrestricted shell execution should not be required for the planning use case.

## Session and thread identity

Persist enough identity to distinguish:

- Task Contract id;
- controller run;
- worker run;
- native agent session/thread;
- worktree;
- base SHA;
- resulting head SHA.

This is necessary for resume/fork/cancel/review without confusing sessions.

## Usage metadata

When an agent exposes token/cost/usage data, record it as optional execution metadata. It must never be treated as completion evidence, but it can support later optimization of which worker performs which task.

## Bridge security

Plan bridge acceptance should eventually test:

- loopback default;
- allowed roots;
- blocked sensitive paths;
- tunnel host validation;
- auth mode;
- token/secret redaction;
- restricted tool profile;
- no remote unrestricted worker launch by default.

---

# Capability-to-reference matrix

| Capability | Primary references | Secondary references | Target decision |
|---|---|---|---|
| Codex -> Claude full agent delegation | `claude-code-agent-for-codex` | `mcp-agents`, `claude-codex-bridge` | Adapt behind project Worker Contract |
| Multi-harness fidelity / native events | `codex-host` | ACP, `codex-acp` | Add capability descriptors + native extension events |
| Generic agent protocol | ACP | `codex-acp`, `mcp-agents` | Reference/compare; do not replace Task/Evidence contracts |
| Bidirectional/recursive delegation safety | `claude-codex-bridge` | `mcp-agents` | Add delegation-depth/cycle guards |
| Async worker lifecycle | `claude-code-agent-for-codex`, `mcp-agents` | `codex-host` | Normalize start/status/result/cancel |
| Official Antigravity worker | `codex-antigravity-subagent` | `mcp-agents` | Official `agy` path, no gateway dependency |
| ChatGPT Web repo context | `codexbridge` | `codex-chatgpt-bridge`, `bifrost` | Bounded workspace/read-mostly profile |
| Plan artifact handoff | `codexbridge`, `bifrost` | `codex-chatgpt-bridge` | `.ai-control/` revisioned artifacts |
| ChatGPT MCP/App contract | OpenAI Apps SDK examples | CodexBridge implementations | Follow official contract/annotations where applicable |
| Tunnel/auth hardening | `codex-gpt-bridge` | `codexbridge`, `codex-chatgpt-bridge` | Loopback + roots + host/auth policy |
| Worktree-per-task governance | This project | Git native worktree primitives | Build locally, project-owned |
| Task Contract | This project | — | Authoritative local schema |
| Evidence Bundle | This project | — | Authoritative local schema |
| Exact-head acceptance | This project | Codex review mechanisms | Authoritative local gate |
| Drift/dependency control | This project | bridge recursion patterns as partial input | Build locally |

---

# Initial build-vs-reuse decision

## Reuse/adapt at interface level

- Claude `-p` invocation/job patterns;
- generic worker job/progress/cancel semantics;
- official `agy` invocation patterns;
- CodexHost capability/event fidelity concepts;
- ACP capability negotiation/event taxonomy;
- ChatGPT MCP repository-context and plan-handoff patterns;
- loopback/tunnel/auth operational hardening.

## Build locally and keep authoritative

- Task Contract schema;
- Evidence Bundle schema;
- Acceptance Result schema;
- plan revision semantics;
- one-task-one-worktree manager;
- base/head SHA checks;
- scope overlap/dependency graph;
- delegation graph/cycle policy;
- drift detection;
- Codex acceptance policy;
- post-merge verification policy.

## Do not adopt

- hidden provider/account switching;
- quota/cooldown evasion;
- dangerous permission bypass as a default;
- shared mutable worktrees for concurrent tasks;
- worker narrative as completion evidence;
- remote planning tools directly owning unrestricted local execution;
- automatic replay of unknown-outcome implementation turns;
- a lowest-common-denominator adapter that discards useful native worker events.
