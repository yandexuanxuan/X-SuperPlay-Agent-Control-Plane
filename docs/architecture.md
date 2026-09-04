# Architecture

## Purpose

X-SuperPlay Agent Control Plane coordinates planning, bounded task dispatch, isolated CLI workers, evidence collection, and acceptance.

It is deliberately separate from any model/API gateway. Worker model access may be native/direct or routed through an external gateway, but the control plane must remain usable without such a gateway.

## Plane model

```text
Planning Plane   - ChatGPT Web
Control Plane    - Codex controller/reviewer
Execution Plane  - Claude Code / Antigravity / future CLI workers
Evidence Plane   - Task/Evidence/Acceptance artifacts + Git identity
Survival Plane   - native Codex / Claude Code / official Antigravity CLI
```

The first four planes may improve automation. The Survival Plane is the escape hatch and may not depend on this project.

## Main workflow

```text
User requirement
      |
      v
ChatGPT Web planning
      |
      v
.ai-control/current-plan.md
.ai-control/acceptance.yaml
      |
      v
Codex controller
      |
      +--> validate plan revision
      +--> inspect repository/base SHA
      +--> create bounded Task Contracts
      +--> check path/dependency conflicts
      |
      +-------------------+-------------------+
      |                                       |
      v                                       v
Claude Code worker                      Antigravity worker
      |                                       |
      v                                       v
isolated task worktree                 isolated task worktree
      |                                       |
      +-------------------+-------------------+
                          |
                          v
                    Evidence Bundle
                          |
                          v
                    Codex reviewer
                          |
                +---------+---------+
                |                   |
                v                   v
              PASS                REJECT
                |                   |
                v                   v
            promote           bounded remediation
                |
                v
        post-merge exact-head retest
```

## Planning artifacts

Project-local planning state should use a bounded, auditable artifact directory:

```text
.ai-control/
├── intent.md
├── current-plan.md
├── acceptance.yaml
├── state.json
├── decisions/
│   └── ADR-*.md
├── tasks/
│   ├── TASK-001.yaml
│   └── TASK-002.yaml
└── evidence/
    ├── TASK-001/
    └── TASK-002/
```

### `intent.md`

Stable user objective and explicit non-goals.

### `current-plan.md`

Authoritative current plan. Every material revision must have a revision id and reason.

### `acceptance.yaml`

Machine-readable completion gates.

### `state.json`

Current control-plane state: plan revision, tasks, worker assignment, review state. It must not contain secrets.

### `tasks/`

Immutable or revision-tracked Task Contracts.

### `evidence/`

Evidence required for acceptance. Full LLM transcripts are optional diagnostics, not the primary proof of completion.

## Role model

### ChatGPT Web — Architect / Planner

Responsibilities:

- clarify intent and constraints;
- reason over selected repository context;
- produce architecture/implementation plans;
- define/refine acceptance criteria;
- review summarized evidence and architecture drift;
- avoid acting as the high-frequency local executor.

The bridge should expose only controlled repository context and planning artifacts. It should not grant unrestricted machine control.

### Codex — Controller / Acceptance Reviewer

Responsibilities:

- consume the exact plan revision;
- decompose work into bounded Task Contracts;
- assign workers;
- enforce worktree/scope/dependency isolation;
- observe/cancel worker runs;
- collect evidence;
- independently verify scope, behavior, regressions, and Git identity;
- return PASS/REJECT and bounded remediation;
- promote only accepted artifacts.

### Claude Code — Primary Implementation Worker

Responsibilities:

- execute one Task Contract in one assigned worktree;
- inspect/edit/run authorized tools/tests;
- remain within declared scope;
- report blockers without expanding the task;
- emit structured evidence.

The intended adapter wraps the full Claude Code agent loop via its official non-interactive CLI behavior rather than exposing only low-level tool primitives.

### Antigravity — Secondary Independent Worker

Responsibilities may include:

- independent investigation;
- second-opinion review;
- alternative implementation;
- research or debugging;
- bounded implementation when explicitly allowed.

Preferred adapter launches the locally installed/authenticated official `agy` CLI. It must not require a subscription-proxy path.

## Worker contract

Every adapter normalizes the same conceptual lifecycle:

```text
prepare(task, worktree) -> run_id
run(run_id) -> queued | launching | running | collecting | completed | failed | cancelled
cancel(run_id)
collect(run_id) -> Evidence Bundle
```

Minimum worker metadata:

- worker kind;
- executable path/identity;
- version;
- task id;
- worktree path;
- base SHA;
- start/end timestamps;
- process/result status;
- bounded log references;
- evidence path.

## Worktree isolation

Target:

```text
repo/
.worktrees/
├── TASK-001/
├── TASK-002/
└── TASK-003/
```

Required invariants:

```text
one active task -> one worktree
one worktree -> one owning task
task is pinned to declared base_sha
worker cannot silently change base identity
acceptance checks actual changed paths against allowed scope
```

Parallel execution is allowed only when scopes/dependencies are compatible. Overlapping tasks must be serialized or explicitly ordered.

## Evidence-driven acceptance

Acceptance must evaluate:

1. **Scope** — only allowed changes occurred.
2. **Behavior** — required tests/assertions pass.
3. **Regression safety** — relevant pre-existing behavior remains valid.
4. **Artifact identity** — review targets the exact `head_sha` being promoted.
5. **Plan freshness** — the task still corresponds to the current plan revision.
6. **Unresolved risk** — blockers are absent or explicitly governed.

A worker exit code of zero is process evidence, not completion evidence.

## Drift control

A worker may not silently turn one task into an expanding chain of new goals.

If new work is discovered:

```text
worker reports blocker/follow-up
        |
        v
controller decides necessity
        |
   +----+----+
   |         |
 needed   not needed
   |         |
new task/   out of scope
revision
```

REJECT remediation must remain bounded to the original objective unless a new plan revision explicitly authorizes a scope change.

## Failure-domain isolation

Required behavior:

```text
if this control plane fails:
    native Codex remains usable
    native Claude Code remains usable
    official agy remains usable

if one worker fails:
    controller and independent workers remain usable

if an optional external model gateway fails:
    native/direct workers remain usable
```

## Security boundaries

- no API keys/OAuth/session state in Git;
- worktree roots are validated/canonicalized;
- worker commands are built without unsafe shell interpolation of untrusted task fields;
- permission tiers/allowlists should be explicit where the CLI supports them;
- destructive permission-bypass modes are not the default worker path;
- execution logs are bounded and secrets must be redacted/excluded;
- provider/account switching must be explicit, not hidden;
- no account rotation or quota/cooldown evasion.

## External integration boundary

This project may integrate with a separately maintained model-access gateway, but only through an explicit adapter/configuration boundary. Model/provider routing, credential namespaces, API health, and subscription proxying do not belong in this repository.
