# Reference Projects and Adoption Matrix

Status: **authoritative research/reference inventory for initial implementation**

The projects below are design references. They are not automatically vendored dependencies.

Before copying code or taking a runtime dependency, re-check the current repository state, license, release cadence, security model, and whether a smaller interface-level adaptation is sufficient.

## Adoption rule

```text
reference != dependency
```

For every proposed import:

1. identify the exact capability needed;
2. verify current license and provenance;
3. prefer protocol/interface compatibility over vendoring;
4. pin the exact upstream identity if code is imported;
5. record modifications and update policy;
6. retain this project's Task/Evidence/Acceptance contracts as the authoritative boundary;
7. preserve native-tool independence.

---

## 1. Qihao-Duan/claude-code-agent-for-codex

Repository:

- https://github.com/Qihao-Duan/claude-code-agent-for-codex

### Why it matters

This project exposes **Claude Code's full agent loop** to Codex over MCP by wrapping `claude -p`, rather than mirroring individual Read/Edit/Bash primitives.

Its core direction is:

```text
Codex -> MCP -> bridge -> claude -p -> Claude Code agent loop
```

### Patterns to evaluate/adopt

- treating Claude Code as a real autonomous worker, not merely a Claude model endpoint;
- synchronous mode only for small bounded tasks;
- async job APIs for real coding tasks;
- durable job ids and persisted job state;
- progress/heartbeat reporting;
- cancellation;
- structured distinction between launch failure, running, timeout, parsing, and worker failure;
- explicit working directory;
- runtime profiles that limit inherited local state;
- permission tiers/deny lists;
- bounded output/log paths.

### What not to inherit blindly

- its exact MCP schema does not replace this project's Task Contract/Evidence Bundle;
- a Claude process completing successfully is not sufficient for acceptance;
- worker state must additionally bind to worktree ownership, base SHA, plan revision, changed-path scope, and exact-head review;
- local permission defaults must be reviewed against this project's threat model.

### Target use here

Primary reference for:

```text
workers/claude-code/
```

Priority: **HIGH**.

---

## 2. thomaswitt/mcp-agents

Repository:

- https://github.com/thomaswitt/mcp-agents

### Why it matters

`mcp-agents` provides a common MCP-style delegation surface across multiple CLI agents, including Codex, Claude Code, and an Antigravity-backed provider. It demonstrates how a stable wrapper can hide provider-specific execution mechanics from the caller.

### Patterns to evaluate/adopt

- one normalized integration pattern across heterogeneous agents;
- provider/worker adapter abstraction;
- long-running job lifecycle;
- `start -> status -> result -> cancel` patterns;
- bounded result paging;
- liveness/heartbeat separation from final output;
- durable/resumable controller-side state where appropriate;
- no automatic fallback between semantically different providers;
- keeping wrapper-owned interfaces stable while underlying CLI integrations evolve.

### Important current lesson

Its Codex path has moved toward `codex app-server` internally while keeping MCP at the wrapper boundary. This is a useful architectural lesson: this project should own its **worker contract**, not expose an upstream CLI's unstable/private protocol as the project contract.

### What not to inherit blindly

- this project should not become a generic model/provider router;
- this project does not need every provider supported by `mcp-agents` initially;
- remote browser/provider features are outside the initial scope;
- upstream durability/state semantics must not override our Git worktree and exact-head governance.

### Target use here

Reference for:

```text
worker adapter interface
controller job lifecycle
progress/cancel/result normalization
future generic CLI workers
```

Priority: **HIGH**.

---

## 3. IlleJiViN/codex-antigravity-subagent

Repository:

- https://github.com/IlleJiViN/codex-antigravity-subagent

### Why it matters

This project lets Codex use the **locally authenticated official Antigravity CLI (`agy`)** as an external agent. Its default behavior favors bounded delegation and plan mode instead of attempting to convert a subscription into a generic model API.

### Patterns to evaluate/adopt

- `agy` availability/version check before dispatch;
- official CLI/headless process as the worker boundary;
- locally owned authentication rather than credential extraction;
- plan/read-only default where appropriate;
- explicit edit-capable mode;
- no exposed dangerous permission-bypass flags;
- bounded execution timeout and output size;
- Codex remains responsible for verification and final acceptance;
- no need for a model subscription proxy to launch the worker.

### What not to inherit blindly

- returning an Antigravity textual answer is not enough for implementation acceptance;
- this project additionally requires task worktrees, base/head SHA evidence, changed-path checks, tests, and exact-head review;
- account/session state must stay outside task artifacts and Git.

### Target use here

Primary reference for:

```text
workers/antigravity/
```

Priority: **HIGH**.

---

## 4. naplesblue/codexbridge

Repository:

- https://github.com/naplesblue/codexbridge

### Why it matters

CodexBridge connects ChatGPT Web/Developer Mode to bounded local repository context through MCP and supports an artifact-based handoff such as `.ai-bridge/current-plan.md` to a local execution agent.

It explicitly separates:

```text
ChatGPT writes/updates a handoff artifact
             |
             v
user-started local watcher/executor
             |
             v
local agent execution
```

rather than making a remote ChatGPT tool silently execute unrestricted local agents.

### Patterns to evaluate/adopt

- bounded repository opening/context selection;
- read-mostly/default-minimal tool surfaces;
- workspace-scoped writes;
- blocked secret/cache/build paths;
- plan handoff as a file/artifact, not a copied full transcript;
- local watcher separated from remote MCP tools;
- content-hash/revision detection to avoid duplicate execution;
- compact status/diff/evidence results;
- operation journal/audit concepts;
- explicit statement that bridge behavior does not bypass product limits.

### What not to inherit blindly

- this project should not turn ChatGPT Web into the implementation worker by default;
- broad local shell/edit permissions are unnecessary for the first planning bridge;
- our authoritative artifact namespace is `.ai-control/`, not `.ai-bridge/`;
- local execution must flow through Codex controller/task contracts rather than directly from a plan watcher to an unrestricted worker.

### Target use here

Primary reference for:

```text
bridges/chatgpt-web/
.ai-control/ handoff semantics
plan revision detection
bounded repository context
```

Priority: **HIGH**.

---

## 5. Dalomeve/codex-chatgpt-bridge

Repository:

- https://github.com/Dalomeve/codex-chatgpt-bridge

### Why it matters

Additional reference for bridging ChatGPT-side planning/context with local Codex-oriented execution.

### Patterns to evaluate

- local bridge topology;
- repository context exchange;
- ChatGPT-to-Codex handoff ergonomics;
- small operational surface for a personal/local development setup.

### Target use here

Secondary comparison source for the ChatGPT planning bridge.

Priority: **MEDIUM**.

---

# Capability-to-reference matrix

| Capability | Primary reference | Secondary reference | Decision |
|---|---|---|---|
| Codex -> Claude Code full agent delegation | `claude-code-agent-for-codex` | `mcp-agents` | Adapt pattern behind our Worker Contract |
| Async worker lifecycle | `claude-code-agent-for-codex` | `mcp-agents` | Adopt normalized start/status/result/cancel semantics |
| Generic multi-agent adapter surface | `mcp-agents` | — | Adopt concept, keep initial providers minimal |
| Codex -> official Antigravity CLI | `codex-antigravity-subagent` | `mcp-agents` | Adapt official-CLI delegation pattern |
| ChatGPT Web local repo context | `codexbridge` | `codex-chatgpt-bridge` | Adapt bounded read/context model |
| Plan artifact handoff | `codexbridge` | `codex-chatgpt-bridge` | Adopt artifact/revision concept using `.ai-control/` |
| Worktree-per-task governance | This project | Git native worktree primitives | Build here; references do not define our acceptance model |
| Task Contract | This project | — | Authoritative local schema |
| Evidence Bundle | This project | — | Authoritative local schema |
| Exact-head acceptance | This project | Codex review mechanisms where useful | Authoritative local gate |
| Task/review drift control | This project | — | Build here |

# Initial build-vs-reuse decision

## Reuse/adapt at interface level

Strong candidates:

- Claude `-p` process invocation patterns;
- worker job/progress/cancel normalization;
- official `agy` invocation patterns;
- ChatGPT MCP repository-context and handoff concepts.

## Build locally

Must remain project-owned:

- Task Contract schema;
- Evidence Bundle schema;
- Acceptance Result schema;
- plan revision semantics;
- one-task-one-worktree manager;
- base/head SHA checks;
- scope overlap/dependency graph;
- drift detection;
- Codex acceptance policy;
- post-merge verification policy.

## Do not adopt

- hidden provider/account switching;
- quota/cooldown evasion;
- dangerous permission bypass as a default;
- shared mutable worktrees for concurrent tasks;
- worker narrative as completion evidence;
- remote planning tools directly owning unrestricted local execution.
