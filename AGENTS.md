# AGENTS.md

Canonical, vendor-neutral agent context for this repository, per [ENG-0006](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0006-agentic-primitives-governance.md).

## What this repository is

The `inventory` capability of the `qwts` fleet: the reusable `dependency-inventory` workflow, the zero-dependency normalizer behind it, and the weekly fleet catalog. Extracted from [agent-sop](https://github.com/qwts/agent-sop) under the fleet split (qwts/agent-sop#371, #372). Map: [README.md](README.md).

## Shared agent conventions

PR-first workflow, validation before push, commit and PR hygiene, and the untrusted-input threat model are defined once, for every repo, in the [org-wide agent conventions](https://github.com/qwts/agent-sop/blob/main/docs/reference/agent-conventions.md). This repository is governed by agent-sop: its shared SOPs and engineering decisions apply here by default ([ENG-0008](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0008-shared-sop-inheritance.md)).

## What is specific to this repository

- **Zero runtime dependencies** ([ENG-0004](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0004-centralize-shared-cicd.md)): the normalizer runs from a bare checkout inside consumer repos, so `tools/dependency-inventory/` imports nothing outside `node:` builtins. `markdownlint-cli` is the only dev dependency.
- **Consumer contract:** consumers pin [`.github/workflows/dependency-inventory.yml`](.github/workflows/dependency-inventory.yml) at a commit SHA recorded in [qwts-agent-org](https://github.com/qwts/qwts-agent-org). Renaming or removing an input, changing a default, or changing the inventory schema (`qwts.dependency-inventory/v1`) breaks every consumer: say so in the PR.
- **Gates before a PR:** `npm test`, `npm run lint:markdown`, and docs-gov over this repo's docs (the tool lives in [agent-sop](https://github.com/qwts/agent-sop/tree/main/tools/docs-gov); this repo's config is [`docs-gov.config.json`](docs-gov.config.json)). A new doc must be linked from [README.md](README.md), or the `orphan-doc` rule fails it.
- **Report-only** ([ENG-0015](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0015-dependency-inventory.md)): the workflow never fails a consumer's build. Enforcement stays with osv-scanner and cargo-deny; do not add gates here.
