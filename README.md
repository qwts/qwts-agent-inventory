# qwts-agent-inventory

The `inventory` capability of the `qwts` engineering fleet: a report-only inventory of what each governed repository depends on and which tools it runs, delivered as a reusable GitHub Actions workflow, plus the weekly cross-repo catalog built from it. Nothing here fails a build; license and advisory enforcement stay with osv-scanner and cargo-deny.

Extracted from [qwts/agent-sop](https://github.com/qwts/agent-sop) at commit `ed5c5d8` under the fleet split ([qwts/agent-sop#371](https://github.com/qwts/agent-sop/issues/371), [qwts/agent-sop#372](https://github.com/qwts/agent-sop/issues/372)): agent-sop keeps the rules, procedures, and guides; each capability lives in its own repository and is consumed by SHA-pinned reference.

## What is here

- [`.github/workflows/dependency-inventory.yml`](.github/workflows/dependency-inventory.yml) — the reusable workflow consumers call: a Syft SBOM, then the normalizer, then one `dependency-inventory` artifact.
- [`tools/dependency-inventory/`](tools/dependency-inventory/inventory.mjs) — the zero-dependency normalizer: CLI, library, tests, and fixtures.
- [`.github/workflows/inventory-catalog.yml`](.github/workflows/inventory-catalog.yml) — the scheduled fleet catalog.
- [`inventory-defaults.config.json`](inventory-defaults.config.json) — the fallback config the catalog applies to a fleet repository that has not committed its own.
- [`dependency-inventory.config.json`](dependency-inventory.config.json) — this repository's own inventory config.

## Adopting the workflow in a repository

Call the workflow pinned to a full commit SHA of this repository, and pass the same SHA as `tooling-ref` so the normalizer runs at the revision you pinned (the input has no default; an empty value fails the job):

```yaml
jobs:
  dependency-inventory:
    uses: qwts/qwts-agent-inventory/.github/workflows/dependency-inventory.yml@<40-hex-sha>
    with:
      tooling-ref: <40-hex-sha>
```

Inputs: `config` (path to the config file, default `dependency-inventory.config.json`); `tooling-ref` (commit SHA of this repository to take the normalizer from; required unless `use-local-tooling` is true, an empty value fails closed); `runs-on` (JSON array of runner labels, default `["ubuntu-latest"]`); `use-local-tooling` (run the normalizer from the caller's own checkout, default `false`); `fail-on-error` (fail the job when Syft or the normalizer errors, default `false` — the inventory is report-only).

Commit a `dependency-inventory.config.json` at the repository root naming what counts:

```json
{
  "manifests": { "npm": ["package.json"], "cargo": ["Cargo.toml"] },
  "workflows": [".github/workflows/*.yml", ".github/workflows/*.yaml"],
  "toolConfigs": [".markdownlint.json", "tsconfig*.json", "deny.toml"]
}
```

`manifests` lists the direct-dependency manifests whose dev dependencies classify a package as dev tooling; `workflows` and `toolConfigs` are globs for the workflow files and tool-config files to report. The reusable workflow needs the file (the normalizer step errors without one); the fleet catalog falls back to `inventory-defaults.config.json`.

Run the normalizer locally against any CycloneDX SBOM:

```bash
node tools/dependency-inventory/inventory.mjs --sbom <sbom.json> --root . --repo <name> --report dependency-inventory.json
```

`npm run inventory` invokes the same CLI; `--aggregate <dir>` folds several per-repo reports into one catalog.

## The weekly fleet catalog

[`inventory-catalog.yml`](.github/workflows/inventory-catalog.yml) runs Mondays 06:00 UTC and on demand, on `ubuntu-latest`. It reads the active governed repositories from the fleet manifest, [governance/repos.json](https://github.com/qwts/qwts-agent-org/blob/main/governance/repos.json) in qwts-agent-org, inventories each with this repository's own tooling, and uploads one `fleet-catalog` artifact (JSON plus Markdown). Prerequisite: a `FLEET_INVENTORY_TOKEN` secret with read access to qwts-agent-org and to the governed repositories.

## Where the pin is recorded

Consumers do not track this repository's default branch. The SHA the fleet consumes is recorded as the `inventory` capability in qwts-agent-org's `org.json`, next to the other capability repositories; moving the fleet to a new revision is a change there, not here.

## Documentation

- [Dependency & tooling inventory](docs/dependency-inventory.md) — how it works, the inventory schema, consuming the workflow, the catalog, and the SHA-pin asymmetry.
- [AGENTS.md](AGENTS.md) — agent context for this repository.

## Development

```bash
npm ci
npm test               # node --test over the normalizer's tests
npm run lint:markdown  # markdownlint over docs/, AGENTS.md, README.md
```

The docs are governed by agent-sop's docs-gov tool against [`docs-gov.config.json`](docs-gov.config.json); run `node <agent-sop checkout>/tools/docs-gov/docs-gov.mjs` from this repository's root. CI ([`ci.yml`](.github/workflows/ci.yml)) runs the tests, a smoke run of the normalizer against this repository's config, and the Markdown lint.

## Governance

- The decision record, [ENG-0015](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0015-dependency-inventory.md), stays in agent-sop with the rest of the ENG series.
- The [SOP inventory and migration map](https://github.com/qwts/agent-sop/blob/main/docs/sop/inventory.md) stays in agent-sop too; it is the ENG-0008 record of where each repository's procedures live, not a procedure for this tool.
- The org-wide [agent conventions](https://github.com/qwts/agent-sop/blob/main/docs/reference/agent-conventions.md) apply here.
