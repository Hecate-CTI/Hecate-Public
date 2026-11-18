# Hecate-Public

The shared Hecate catalog contains hunt definitions, candidate detection content,
research and starter knowledge. A catalog entry or merged PR does not prove live
detection coverage in your environment.

## Repository responsibilities

[Core](https://github.com/Hecate-Org/Hecate-Core) supplies the engine and
`hecate-agent`; [Runner](https://github.com/Hecate-Org/Hecate-Runner) supplies
orchestration, integrations, approvals and `hecate-runner`. Public supplies content.
Use a private workspace seeded with reviewed catalog content for enterprise use.

| Directory | Contents |
| --- | --- |
| `hunts/` | LOCK-format records (`H-XXXX.md`) |
| `queries/` | Sigma YAML and native `.spl`, `.kql`, `.eql`, `.cql`, `.leql`, `.s1ql` artifacts |
| `research/` | Research dossiers (`R-XXXX.md`) |
| `knowledge/` | Starter environment profile and hunting guidance |
| `scripts/`, `.github/workflows/` | Review, promotion and deployment checks |

`.cql` is CrowdStrike LogScale; `.leql` is Rapid7 LEQL.

## Install and browse

Use Python 3.11.5+, Git, uv and Bash on Linux/macOS or WSL:

```bash
mkdir hecate-dev
cd hecate-dev
git clone https://github.com/Hecate-Org/Hecate-Core.git
git clone https://github.com/Hecate-Org/Hecate-Runner.git
git clone https://github.com/Hecate-Org/Hecate-Public.git
uv venv .venv
source .venv/bin/activate
uv pip install -e './Hecate-Core[all,dev]' -e './Hecate-Runner[dev]'
export HECATE_WORKSPACE="$PWD/Hecate-Public"
export HECATE_BIN="$PWD/.venv/bin/hecate-agent"
cd Hecate-Public
hecate-runner demo
hecate-agent hunt list
```

The demo uses a temporary workspace, needs no keys and adds no hunt to Public.
For persistent shell setup, see [Runner onboarding](https://github.com/Hecate-Org/Hecate-Runner/blob/main/docs/ONBOARDING.md).
The organization `.github` repository is optional to clone.

## Select knowledge, credentials and output locations

Export `HECATE_WORKSPACE` before either CLI. Both use that workspace's `.env`;
existing environment values win and a cwd `.env` can override workspace values.
Installation creates no `.env` symlink or working API keys. Copy `.env.example`
only when `.env` does not already exist, and configure only enabled features.

Complete `knowledge/environment.md` with verified sources, fields, coverage and
retention. The public profile is an **unverified template**. Adapt
`knowledge/hunting-knowledge.md` to local guidance; `HECATE_HUNTING_KNOWLEDGE=false`
disables that optional guidance without removing evidence or execution gates.

Keep credentials, `.hecate/`, logs, private evidence and findings out of public
commits. Point `HECATE_WORKSPACE` to your private workspace for live use; applications
do not automatically merge it with Public.

## Run catalog hunts without generating new H files

Configure a query provider in the private workspace and use an existing hunt ID:

```bash
hecate-agent context --hunt H-0001 --format json
hecate-agent hunt update H-0001 --status active
hecate-runner sweep stage --hunt-id H-0001 --days 1
hecate-runner sweep pending
hecate-runner sweep approve "<SWEEP-ID>"
```

Keep `HECATE_RUNNER_QUERY_PROVIDER=mock` for practice and
`HECATE_AUTO_HUNT_APPROVAL_REQUIRED=true` while verifying mappings. Continuous
scheduling uses `HECATE_AUTO_HUNT_EXECUTION_ENABLED` and `hecate-runner sweep daemon`.
CTI intake and new-hunt generation are separate; optional analysis, pivots and
tuning may use models.

## Publication and validation boundaries

Generated hunts start in planning. Publication, deployment, execution and BAS have
separate opt-ins. Exact public snapshots require privacy/redistribution review.
Synthetic replay, local BAS matches, smoke checks, native search matches and
correlated fired alerts remain distinct evidence. See [BAS boundaries](https://github.com/Hecate-Org/Hecate-Runner/blob/main/docs/BAS_AUTOMATION.md)
and [release controls](https://github.com/Hecate-Org/Hecate-Runner/blob/main/docs/COMMUNITY_RELEASES.md).

## GitHub Actions deployment settings

Review `.github/workflows/siem-*.yml` before adding repository credentials. They use
GitHub secrets/variables, not local `.env` flags. With endpoint/token configured,
the deploy workflow can push saved queries on relevant `main` updates. Missing
credentials or explicit workflow dry-run selects preview. Review triggers,
permissions and destination namespaces before enabling live access.

This checkout has no `catalog/catalog.yaml` or community-release workflow.
Runner's optional release tooling needs an explicit reviewed catalog and separate
publication workflow; ordinary hunt/query content is not a release bundle.

Review [LICENSE](LICENSE), [LICENSES.md](LICENSES.md) and source attribution before
redistributing imported content.
