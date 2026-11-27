# Environment Profile — Verified Lab Profile

**Profile status:** verified-lab-profile
**Last verified:** 2026-11-27
**Owner:** Hecate Detection & Threat Hunting Team
**Review cadence:** Monthly; recheck after sensor, schema, or simulation changes.

This environment profile establishes the locally verified configuration, active indices,
sourcetypes, and field mappings for the threat hunting estate and validation lab.
Credentials are kept in the untracked `.env`.

## Workspace ownership and source of truth

The selected workspace owns this file. Core supplies the readers and Runner supplies
execution. Set `HECATE_WORKSPACE` to the absolute adopter workspace path when launching
from elsewhere.

## Asset and technology scope

| Scope | Verified value | Verification source / date / owner |
|---|---|---|
| Organization / tenant / account boundary | Hecate Detection & Threat Hunting Lab | Verified / 2026-11-27 / Hunting Lead |
| Included hosts, groups and critical assets | `fw-edge-01`, `ai-worker-01`, `salt-master-01`, `web-drupal-01`, `mac-workstation-09`, `192.168.65.1` | Splunk events & BAS verification / 2026-11-27 |
| Operating systems / versions | Linux (Ubuntu 24.04 LTS / Debian container base), macOS 15 (Sequoia / Mach-O) | Sensor telemetry / 2026-11-27 |
| Applications / exposed services | PAN-OS GlobalProtect (`hipreport.esp`), GPUStack AI (`/v2/inference-backends`), SaltStack / HPE Juniper SSR Conductor, Drupal Core (`/jsonapi/`) | Service telemetry / 2026-11-27 |
| Cloud / SaaS / identity platforms | Splunk Enterprise (`https://localhost:8089`), Docker / Container runtime | Ingestion API / 2026-11-27 |
| Exclusions and prohibited actions | Production destructive commands; out-of-scope non-lab subnets | Execution policy / 2026-11-27 |
| Threat priorities and business impact | Initial access exploitation (CVE-2024-3400, CVE-2026-102895, CVE-2026-103406, CVE-2026-9082), macOS ClickFix infostealers | Threat matrix / 2026-11-27 |

## Telemetry availability contract

Use **verified**, **partial**, **unavailable** or **unknown** for each source.

| Source / event family | Status | Index/table + provider/channel | Field mapping | Asset/tenant coverage | Retention + ingestion lag | Evidence / date / owner |
|---|---|---|---|---|---|---|
| Endpoint process and lineage | verified | `index=main`, `index=hecate_synthetic`<br/>`sourcetype=hecate:telemetry` | `ParentImage`, `ParentCommandLine`, `Image`, `CommandLine`, `User`, `ProcessId`, `host` | Linux servers, GPU nodes, macOS endpoints | 30 days retention, < 5s ingestion lag | Splunk REST API verified / 2026-11-27 / Hunting Team |
| Authentication / identity / Kerberos | partial | `index=main`<br/>`sourcetype=hecate:telemetry` | `User`, `action`, `EventCode`, `src_user` | Linux & container hosts | 30 days retention, < 10s lag | Splunk audit / 2026-11-27 |
| File events / persistence | verified | `index=main`, `index=hecate_synthetic`<br/>`sourcetype=hecate:telemetry` | `TargetFilename`, `Image`, `CommandLine`, `User`, `host` | macOS LaunchAgents (`/Library/LaunchAgents/*`), Linux `/etc/cron.d/*` | 30 days retention, < 5s lag | Splunk REST API verified / 2026-11-27 / Hunting Team |
| Registry events / persistence | unavailable | N/A (non-Windows target estate) | N/A | Windows endpoints not deployed in primary tier | N/A | Environment audit / 2026-11-27 |
| Network connections / DNS / proxy | verified | `index=main`, `index=hecate_synthetic`<br/>`sourcetype=hecate:telemetry` | `QueryName`, `DestinationIp`, `DestinationPort`, `SourceIp`, `SourcePort`, `Initiated`, `Protocol` | macOS DNS resolver, edge egress gateways | 30 days retention, < 5s lag | Splunk REST API verified / 2026-11-27 / Hunting Team |
| Application / web server | verified | `index=main`, `index=hecate_synthetic`<br/>`sourcetype=hecate:telemetry` | `cs-uri-stem`, `cs-uri-query`, `cs-method`, `cs-cookie`, `c-ip`, `sc-status` | GlobalProtect gateway, GPUStack web API, Salt Conductor, Drupal Core web service | 30 days retention, < 5s lag | Splunk REST API verified / 2026-11-27 / Hunting Team |
| Cloud control-plane / data-plane | partial | `index=main`<br/>`sourcetype=hecate:telemetry` | `eventSource`, `eventName`, `userIdentity` | Container orchestration | 30 days retention | Lab audit / 2026-11-27 |
| SaaS audit | unavailable | N/A | N/A | Out of scope for current keeper hunts | N/A | Scope review / 2026-11-27 |

For each verified row, actual provider and sourcetypes are mapped to native schema representations.

## Query access and validation

- **Backend / endpoint reference:** Splunk Enterprise API (`https://localhost:8089`) configured via `.env` (`SPLUNK_HOST`, `SPLUNK_TOKEN`).
- **Permitted indexes/tables and role:** `main`, `hecate_synthetic`; role: `admin` / `hecate-reader`.
- **Known-event control query / expected result:** `index=* | head 1` (returns verified event; non-zero count).
- **Last successful authentication/query check:** 2026-11-27T04:20:00Z.
- **Event counts and required-field population for the relevant window:** Verified populated for `Image`, `ParentImage`, `CommandLine`, `QueryName`, `TargetFilename`, `DestinationIp`, `cs-uri-stem`, `cs-cookie`.
- **Exact vs sampled counts / pagination and result limits:** Exact counts via `eventcount` and `spath`; search limits capped at 100-500 events per sweep.
- **Event time vs ingestion time and maximum observed lag:** Event timestamps synchronized in UTC (`timestamp_utc`); observed ingestion latency < 2 seconds.
- **Query time, row, cost and timeout limits:** Search timeout 300s; memory quota 4GB.

Before live hunting, queries are verified via COUNT-FIRST checks against known event timestamps.

## Baselines and benign explanations

- **Linux / Container Base:** Normal parent process trees originate from `systemd`, `init`, or container entrypoint wrappers. Shell execution (`/bin/sh`, `/bin/bash`, `/usr/bin/id`) directly spawned by daemon processes (`gpustack`, `salt-master`, `pan_comm`, `php-fpm`, `apache2`, `nginx`) is strictly anomalous and represents high-confidence exploitation.
- **macOS Workstations:** Standard DNS lookups target internal domains and legitimate Apple infrastructure. DNS resolution of known campaign staging domains (`profitnow.io`) or egress to non-standard HTTP C2 ports (`:8133`) are anomalous and indicative of infostealer execution.
- **Perimeter Gateways:** Legitimate PAN-OS session cookies contain structured alphanumeric tokens; shell metacharacters (`$(...)`, backticks, command chaining) in `cs-cookie` on `/ssl-vpn/hipreport.esp` have zero known benign explanations.
- **Drupal Core / Web Workloads:** Normal JSON:API requests query structured entity endpoints without nested filter query arrays (`filter[...]`) attempting polymorphic deserialization or invoking CLI shell interpreters.

## Visibility gaps and conclusion limits

- **Detection Boundary:** Verified coverage applies to the active search lookback window (default 1 to 7 days). Events aged beyond the lookback window will yield `clean` outcomes without disproving historical compromise.
- **Telemetry Ingestion Cadence:** Synthetic and Purple Team replays must maintain timestamps within the active evaluation window (`now - window_days`).

## Execution and response authorization

- **Hunt owner / approver / escalation contact:** Hunting Lead (`slack:U0C9TQPLK2P`) / Security Operations.
- **Approved scope and schedule:** Scheduled autonomous sweeps active across keeper hunts (`H-0021`, `H-0030`, `H-0031`, `H-0033`, `H-0036`, `H-0038`, `H-0039`, `H-0041`, `H-0042`).
- **Read-only query / pivot budgets:** 300s query timeout, maximum 3 pivot iterations per triage finding.
- **Containment / BAS / deployment / suppression authorization:** Staged sweep approvals required prior to action taking; notifications routed to designated Slack alert channels.


Existing H files can be reviewed and adapted without enabling generation. Before
scheduled execution, verify their status, query compatibility, scope and approval.
Use the approval and execution controls documented in `.env.example`. A query result
or model verdict cannot authorize a response or widen the scope itself.
