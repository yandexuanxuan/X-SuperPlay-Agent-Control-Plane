# Migration Provenance

This repository was created as the dedicated home for Agent Control Plane work that had temporarily been specified in `yandexuanxuan/codex-proxy-suite`.

## Source

Source repository:

- `yandexuanxuan/codex-proxy-suite`

Source architecture PR:

- PR #25 — `docs: define model access and agent control architecture`

Source merge commit:

```text
70eba44917fcfd9000491dd6ebbd52b54abb0140
```

The source material included:

- Agent Control Plane specification;
- model-access vs agent-control separation decision;
- combined development roadmap;
- Codex -> Claude Code delegation design;
- official Antigravity CLI worker design;
- ChatGPT Web planning-handoff design;
- worktree isolation and evidence-driven acceptance rules;
- reference projects.

## Migration decision

After `X-SuperPlay-Agent-Control-Plane` was created, Agent Control Plane responsibilities were moved here so the model-access repository no longer needs to carry this project's implementation plan or internal architecture.

This repository is the authoritative location for Agent Control Plane development after the migration is merged.

## Adaptation during migration

The migration is not a byte-for-byte copy. Content was reorganized into project-owned documents:

```text
README.md                  product objective / architecture / phases
AGENTS.md                  contributor governance
docs/architecture.md       detailed control-plane design
docs/contracts.md          Task / Evidence / Acceptance semantics
docs/reference-projects.md reference inventory and adoption matrix
docs/roadmap.md            authoritative implementation plan
docs/provenance.md         migration lineage
```

References to model-provider implementation details were intentionally removed. External model gateways are treated only as optional integrations because provider routing belongs outside this repository.

## Authority rule

If old copies of Agent Control Plane material exist elsewhere, this repository's current `main` is authoritative after the migration merge.
