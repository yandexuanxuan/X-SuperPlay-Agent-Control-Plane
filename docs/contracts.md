# Control Contracts

Status: **design contract for implementation**

The control plane must converge on machine-readable contracts. The examples below define semantics; implementation should formalize them under `schemas/` and validate them in tests.

## 1. Task Contract

```yaml
task_id: TASK-003
plan_revision: PLAN-2026-09-05-r1
base_sha: <git-sha>
objective: >-
  Implement the specified bounded change.

worker:
  kind: claude-code
  profile: default

scope:
  allowed_paths:
    - src/provider/**
    - tests/provider/**
  forbidden_paths:
    - .github/**

non_goals:
  - do not change unrelated authentication
  - do not alter unrelated architecture

acceptance:
  commands:
    - <test command>
  assertions:
    - required behavior is observable
    - no changes exist outside allowed scope

execution:
  max_turns: 30
  timeout_policy: bounded
  commit_required: true

expected_evidence:
  - base_sha
  - head_sha
  - changed_files
  - git_diff_summary
  - test_commands
  - test_results
  - unresolved_risks
```

### Required semantics

- `task_id` is immutable for one logical task lineage.
- `plan_revision` pins the task to one planning artifact revision.
- `base_sha` is mandatory and must match the actual task worktree base.
- `allowed_paths` must be bounded; unrestricted root scope requires explicit policy approval.
- `forbidden_paths` overrides allowed globs.
- `non_goals` constrain drift.
- worker cannot modify its own authoritative Task Contract unless the controller explicitly issues a revision.

## 2. Worker Run State

```yaml
worker_run_id: <id>
task_id: TASK-003
worker_kind: claude-code
phase: queued | launching | running | collecting | completed | failed | cancelled
worktree: <canonical path>
base_sha: <sha>
started_at: <timestamp>
last_heartbeat_at: <timestamp>
ended_at: <timestamp | null>
process_exit_code: <int | null>
```

Process success and task acceptance are separate concepts.

## 3. Evidence Bundle

```yaml
task_id: TASK-003
plan_revision: PLAN-2026-09-05-r1
worker_kind: claude-code
worker_run_id: <id>
base_sha: <sha>
head_sha: <sha>
changed_files:
  - path/a
  - path/b
commands_run:
  - <test command>
results:
  - command: <test command>
    exit_code: 0
    summary: passed
scope_check:
  unexpected_files: []
unresolved: []
execution_access:
  mode: native | gateway
  gateway: optional
  provider_id: optional
  access_class: optional
```

### Evidence invariants

- `task_id` must match the Task Contract.
- `plan_revision` must match the dispatched task revision.
- `base_sha` must match the declared base.
- `head_sha` must exist and identify the artifact under review.
- changed files must be independently recomputed from Git; worker-provided list is not trusted by itself.
- required test results must be independently inspectable.
- evidence must not include API keys, OAuth tokens, cookies, or raw credential state.
- full worker transcript is optional diagnostic evidence, never sufficient completion proof.

## 4. Acceptance Result

```yaml
task_id: TASK-003
plan_revision: PLAN-2026-09-05-r1
reviewed_head_sha: <sha>
verdict: PASS | REJECT | BLOCKED
failed_gates: []
required_remediation: []
reviewed_at: <timestamp>
reviewer: codex
```

### Acceptance gates

Codex must verify at least:

```text
scope
behavior
regression safety
artifact identity
plan revision freshness
unresolved blockers
```

A PASS is invalid if `reviewed_head_sha` differs from the artifact promoted later without a new review/retest gate.

## 5. Plan State

Recommended `.ai-control/state.json` concept:

```json
{
  "active_plan_revision": "PLAN-2026-09-05-r1",
  "base_sha": "...",
  "tasks": {
    "TASK-001": {
      "state": "PLANNED",
      "worker": null,
      "worktree": null,
      "reviewed_head_sha": null
    }
  }
}
```

Task state model:

```text
PLANNED
DISPATCHED
RUNNING
EVIDENCE_READY
REVIEWING
PASS | REJECT | BLOCKED | CANCELLED
```

## 6. Revision and drift rules

### Plan drift

If `current-plan.md` changes after dispatch and the change affects a task's objective, scope, dependency, or acceptance gate, the task becomes stale until the controller explicitly revalidates or revises it.

### Task drift

Workers report newly discovered work instead of silently expanding scope. Controller chooses one of:

```text
revise current task
create child task
mark follow-up/out-of-scope
block current task
```

### Review drift

A review is bound to one exact `head_sha`. Any subsequent code change requires re-review of the new identity.

## 7. Parallel dependency rules

Before parallel dispatch, controller must evaluate:

- allowed-path overlap;
- explicit task dependency;
- conflicting base SHAs;
- same worktree assignment;
- generated/shared artifact coupling;
- order-sensitive migrations or lockfiles.

If overlap is unsafe, serialize or reject parallel execution.

## 8. Schema acceptance requirements

When JSON Schemas are implemented:

```text
task_contract_valid_example_pass
missing_task_id_rejected
missing_base_sha_rejected
unbounded_invalid_path_scope_rejected
forbidden_path_override_pass

evidence_bundle_valid_example_pass
evidence_task_mismatch_rejected
evidence_base_sha_mismatch_rejected
evidence_head_sha_required
secret_fields_not_supported

acceptance_result_valid_example_pass
reviewed_head_sha_required
invalid_verdict_rejected
```
