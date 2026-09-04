# Development Roadmap

Status: **authoritative implementation plan**

## Objective

Implement an evidence-driven Agent Control Plane in which ChatGPT Web performs planning/review, Codex controls and accepts work, Claude Code is the primary implementation worker, Antigravity is an independent official-CLI worker, and every task is isolated and reviewable by exact Git identity.

## Requirement set

This repository owns the following requirements:

1. Codex acts as controller and acceptance reviewer.
2. Claude Code performs most implementation work as a real agent loop.
3. Codex can explicitly delegate bounded tasks to Claude Code.
4. Antigravity can be used as an independent worker through official CLI/headless execution.
5. ChatGPT Web performs high-level project planning/review and hands plans to local execution through repository artifacts rather than repeated full-context copy/paste.
6. Parallel workers are isolated by Git worktree and dependency/scope controls.
7. Completion is based on structured evidence and exact-head verification.
8. Failure of this project does not disable native Codex, Claude Code, or official Antigravity CLI.

## Fixed architecture constraints

```text
A. this repository = Agent Control Plane
B. it is not a model/API gateway
C. Codex = controller / dispatcher / acceptance reviewer
D. Claude Code = primary implementation worker
E. official Antigravity CLI = secondary independent worker
F. Task Contract / Evidence Bundle / Acceptance Result are project-owned contracts
G. one active task = one isolated worktree
H. parallel tasks require scope/dependency compatibility
I. worker self-report is never sufficient completion evidence
J. review is bound to exact head SHA
K. plan/task/review drift must be detected
L. native Codex / Claude / agy remain independent
M. external model gateways are optional integrations only
```

Any implementation that violates these constraints requires an explicit superseding architecture decision.

---

# P0 — Governance and architecture bootstrap

Deliverables:

- README product boundary;
- `AGENTS.md` governance;
- architecture document;
- reference-project adoption matrix;
- contracts design;
- authoritative roadmap;
- migration provenance.

### Acceptance

```text
repository_boundary_defined
role_model_defined
reference_inventory_defined
contracts_semantics_defined
roadmap_defined
```

---

# P1 — Machine-readable contracts

Create:

```text
schemas/task-contract.schema.json
schemas/evidence-bundle.schema.json
schemas/acceptance-result.schema.json
```

Add valid/invalid fixtures and deterministic schema tests.

Required Task Contract fields include:

- task id;
- plan revision;
- base SHA;
- objective;
- worker kind/profile;
- allowed/forbidden path scope;
- non-goals;
- acceptance commands/assertions;
- bounded execution policy;
- expected evidence.

### Acceptance

```text
task_contract_schema_pass
evidence_bundle_schema_pass
acceptance_result_schema_pass
missing_task_identity_rejected
missing_base_sha_rejected
invalid_scope_rejected
evidence_task_identity_mismatch_rejected
reviewed_head_sha_required
```

---

# P2 — Worktree and scope manager

Implement one-task-one-worktree lifecycle:

```text
create -> assign -> verify base SHA -> lock ownership -> execute -> inspect -> retain/remove
```

Capabilities:

- canonical worktree paths;
- task ownership metadata;
- base SHA pinning;
- stale/orphan detection;
- safe cleanup;
- changed-path recomputation;
- allowed/forbidden scope validation;
- same-worktree double-assignment rejection.

### Acceptance

```text
one_task_one_worktree_pass
parallel_worktree_isolation_pass
base_sha_pin_pass
stale_worktree_detection_pass
same_worktree_double_assignment_rejected
worker_cannot_silently_change_base_identity
scope_violation_detected
```

---

# P3 — Claude Code worker adapter

Initial primary worker uses official Claude Code non-interactive/headless behavior.

Reference projects:

- `Qihao-Duan/claude-code-agent-for-codex`
- `thomaswitt/mcp-agents`

Required capabilities:

```text
prepare
start
status/progress
cancel
collect
```

Worker launch must receive:

- task id;
- exact worktree cwd;
- bounded task prompt derived from Task Contract;
- appropriate runtime/permission profile;
- bounded timeout/lifecycle policy.

Evidence collector must independently record:

- executable/version identity;
- process status;
- base/head SHA;
- changed paths;
- tests/commands;
- bounded logs;
- unresolved blockers.

### Acceptance

```text
claude_worker_launch_pass
claude_worker_worktree_cwd_pass
claude_worker_bounded_lifecycle_pass
claude_worker_progress_pass
claude_worker_cancel_pass
claude_worker_failure_classification_pass
claude_worker_evidence_pass
claude_worker_scope_violation_detected
claude_exit_zero_not_auto_accept
```

---

# P4 — Codex controller and reviewer

Implement controller state machine:

```text
PLANNED
DISPATCHED
RUNNING
EVIDENCE_READY
REVIEWING
PASS | REJECT | BLOCKED | CANCELLED
```

Controller responsibilities:

- load exact plan revision;
- derive bounded Task Contracts;
- dispatch worker without doing the bulk implementation itself;
- observe/cancel runs;
- collect evidence;
- run independent scope/Git/test checks;
- review exact `head_sha`;
- emit machine-readable Acceptance Result;
- generate bounded remediation on REJECT.

### Acceptance

```text
codex_controller_dispatch_pass
codex_can_delegate_real_claude_agent_loop
codex_does_not_need_to_bulk_implement_delegated_task
codex_exact_head_acceptance_pass
worker_narrative_cannot_override_failed_gate
reject_generates_bounded_remediation
blocked_state_truthful
```

---

# P5 — Parallel dependency and drift control

Before parallel dispatch detect:

- overlapping allowed paths;
- explicit task dependencies;
- conflicting/stale base SHAs;
- same worktree assignment;
- generated/shared-artifact coupling;
- plan revision changes.

Drift categories:

```text
plan drift
task drift
review drift
base identity drift
scope drift
```

### Acceptance

```text
independent_tasks_parallel_pass
overlapping_tasks_serialized_or_rejected
shared_worktree_rejected
dependency_order_enforced
plan_revision_change_invalidates_or_revalidates_dispatch
task_drift_detected
review_drift_detected
base_identity_drift_detected
```

---

# P6 — Antigravity official-CLI worker

Primary reference:

- `IlleJiViN/codex-antigravity-subagent`

Secondary reference:

- `thomaswitt/mcp-agents`

Requirements:

- verify `agy` executable/version;
- use locally authenticated official CLI/headless path;
- default to bounded/read-only or plan behavior where the task does not require edits;
- edit-capable execution only when Task Contract allows it;
- same worktree and Evidence contracts as Claude worker;
- truthful auth/upstream/permission failures;
- no account rotation, quota evasion, or dangerous permission bypass as default behavior.

### Acceptance

```text
agy_worker_check_pass
agy_worker_launch_pass
agy_worker_worktree_isolation_pass
agy_worker_plan_mode_pass
agy_worker_edit_mode_requires_explicit_scope
agy_worker_evidence_pass
agy_worker_auth_failure_truthful
agy_worker_does_not_require_model_gateway
no_account_rotation_evasion
```

---

# P7 — ChatGPT Web planning bridge

Primary reference:

- `naplesblue/codexbridge`

Secondary reference:

- `Dalomeve/codex-chatgpt-bridge`

Initial bridge scope is deliberately narrow.

Target artifacts:

```text
.ai-control/intent.md
.ai-control/current-plan.md
.ai-control/acceptance.yaml
.ai-control/state.json
.ai-control/decisions/
.ai-control/tasks/
.ai-control/evidence/
```

Principles:

- ChatGPT receives selected/bounded repository context;
- bridge defaults to read-mostly;
- writes are restricted to planning/control artifacts initially;
- full conversation transcript is not the source of truth;
- plan has explicit revision/hash;
- Codex consumes exact plan revision;
- execution evidence links back to plan/task identity;
- remote planning bridge does not silently invoke unrestricted local workers;
- local controller remains the execution authority.

### Acceptance

```text
web_can_read_selected_project_context
web_cannot_escape_workspace_scope
web_can_write_or_handoff_plan_artifact
plan_revision_is_stable_and_auditable
codex_consumes_exact_plan_revision
execution_evidence_links_back_to_plan
stale_plan_dispatch_detected
full_chat_transcript_not_required
remote_web_bridge_does_not_directly_bypass_controller
```

---

# P8 — Failure isolation, recovery, and observability

Separate status domains:

```text
Control:
  plan / task / worker / worktree / evidence / acceptance

Worker runtime:
  executable / version / process / heartbeat / logs / failure class

External access (optional metadata only):
  native/direct vs explicitly configured gateway
```

Recovery requirements:

- stale worker detection;
- orphan process handling without killing unrelated processes;
- stale worktree detection;
- recoverable state reconstruction where safe;
- no automatic replay of an unknown-outcome implementation turn;
- native tools remain usable regardless of controller state.

### Acceptance

```text
controller_crash_native_codex_survival_pass
controller_crash_native_claude_survival_pass
controller_crash_official_agy_survival_pass
one_worker_failure_does_not_break_other_workers
stale_worker_detected
orphan_worktree_detected
unknown_outcome_not_auto_replayed
status_does_not_claim_global_health_from_single_component
```

---

# P9 — End-to-end promotion and exact-head verification

End-to-end scenario:

```text
plan revision
   -> Task Contract
   -> isolated worktree
   -> Claude/Antigravity worker
   -> Evidence Bundle
   -> Codex review at exact head
   -> PASS
   -> merge/promote
   -> post-merge exact main retest
```

### Acceptance

```text
end_to_end_claude_delegation_pass
end_to_end_antigravity_delegation_pass
end_to_end_chatgpt_plan_handoff_pass
pre_merge_review_exact_head_pass
post_merge_main_identity_recorded
post_merge_exact_head_retest_pass
completion_report_contains_required_evidence
```

---

# Completion criteria

```text
AGENT_CONTROL_COMPLETE =
    governance_landed
AND task_contract_schema_pass
AND evidence_bundle_schema_pass
AND acceptance_result_schema_pass
AND one_task_one_worktree_pass
AND parallel_worktree_isolation_pass
AND base_sha_pin_pass
AND claude_code_worker_pass
AND codex_controller_dispatch_pass
AND codex_exact_head_acceptance_pass
AND parallel_scope_dependency_control_pass
AND task_and_review_drift_protection_pass
AND antigravity_official_cli_worker_pass
AND chatgpt_web_plan_handoff_pass
AND evidence_not_transcript_is_completion_basis
AND reject_remediation_scope_preserved
AND external_model_gateway_optional
AND native_codex_survival_independent
AND native_claude_survival_independent
AND official_agy_survival_independent
AND post_merge_exact_head_retest_pass
```

## Explicit non-completion states

The project is **not complete** if any of these are true:

- Codex can call a Claude model but cannot delegate to the Claude Code agent loop;
- multiple active workers edit one shared mutable worktree;
- worker exit code or narrative is treated as acceptance;
- task base/head identity is not recorded;
- acceptance reviews a different SHA from the promoted artifact;
- REJECT causes unbounded new work instead of scoped remediation;
- ChatGPT handoff requires manually copying the entire conversation as authoritative context;
- plan revisions can change without invalidating/revalidating in-flight tasks;
- Antigravity worker requires an unofficial subscription proxy;
- control-plane failure disables native Codex/Claude/agy;
- dangerous permission bypass or quota-evasion behavior is required for normal operation.

## Evidence required for final closure

Final acceptance report must include:

- exact main commit SHA;
- implementation file list;
- schema validation output;
- worktree/scope isolation output;
- Claude worker end-to-end evidence;
- Codex exact-head review evidence;
- parallel/drift-control evidence;
- Antigravity official-CLI evidence;
- ChatGPT planning-handoff evidence;
- native-survival evidence;
- CI/local exact-head verification status;
- post-merge retest evidence;
- known limitations/deferred non-goals.

Completion claims without these artifacts are not completion evidence.
