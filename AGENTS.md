# X-SuperPlay Agent Control Plane — contributor guidance

## Product boundary

This repository is an **Agent Control Plane**, not a model gateway.

It owns planning handoff, Codex control/acceptance, CLI worker delegation, Git worktree isolation, structured evidence, and exact-head promotion gates.

It does not own provider API routing, subscription proxying, model aliases, or credential aggregation.

## Governing invariants

1. **Codex controls and verifies; workers implement.** Codex may investigate or perform small bounded edits, but bulk implementation should be delegated when a worker is available.
2. **Agent delegation is not model routing.** `Codex -> Claude model` is not equivalent to `Codex -> Claude Code agent loop`.
3. **One active task, one worktree.** Parallel workers must not share a mutable worktree.
4. **Completion is evidence-driven.** Worker narrative is not acceptance evidence.
5. **Exact artifact identity matters.** Review must bind to the exact returned head SHA and post-merge verification must bind to the promoted mainline SHA.
6. **Task scope may not silently expand.** New work requires an explicit task revision or child task.
7. **Native tools remain independent.** A broken control plane must not disable native Codex, Claude Code, or Antigravity CLI.
8. **No provider quota-evasion logic.** Do not implement account rotation, fingerprint spoofing, permission bypass, or cooldown evasion.
9. **No secrets in Git.** API keys, OAuth state, cookies, session files, local agent profiles, and runtime logs containing secrets are excluded.
10. **Reference does not mean dependency.** Before importing code from a reference project, re-check current license, maintenance state, and the minimum capability actually needed.

## Target control flow

```text
ChatGPT Web plan
      |
      v
Codex controller
      |
      +--> Task Contract --> isolated worktree --> Claude Code worker
      +--> Task Contract --> isolated worktree --> Antigravity worker
      |
      v
Evidence Bundle
      |
      v
Codex exact-head acceptance
      |
      +--> PASS -> promote -> post-merge retest
      +--> REJECT -> bounded remediation
```

## Required contracts

Implementation work must converge on machine-readable contracts for:

- Task Contract;
- Evidence Bundle;
- Acceptance Result;
- plan/control state;
- worker run state.

Schemas belong under `schemas/` and examples/tests must accompany them.

## Worker adapter rule

Every worker should expose the same conceptual lifecycle:

```text
prepare(task, worktree) -> run_id
run(run_id) -> progress/state
cancel(run_id)
collect(run_id) -> Evidence Bundle
```

Worker-specific behavior belongs under `workers/<kind>/`. Controller code must not depend on Claude- or Antigravity-specific output formats once the adapter has normalized them.

## Git/worktree rule

Every task must be pinned to a declared `base_sha`. A worker may not silently rebase or move the base identity. Scope checks must compare actual changed paths with the Task Contract before acceptance.

## Verification rule

A successful worker process exit is not a PASS. Acceptance must evaluate at least:

- scope;
- required behavior/tests;
- regression safety;
- declared base/head identity;
- plan revision freshness;
- unresolved blockers;
- post-merge exact-head verification where required.

## Reference projects

Read `docs/reference-projects.md` before implementing worker bridges or ChatGPT handoff. Do not blindly copy an entire project because one pattern is useful.

## Development order

Follow `docs/roadmap.md`. Contracts and isolation precede autonomous multi-worker orchestration.
