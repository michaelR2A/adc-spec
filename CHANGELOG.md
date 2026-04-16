# Changelog

All notable changes to the ADC specification are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html) from v1.0 onward.
Pre-v1.0 versions are working drafts. Breaking changes may occur between working draft releases.

---

## [0.1.0] — 2026-04-16

### Added

**Core Schema (`spec/adc-v0.1.json`)**
- `adc_version`: spec conformance field
- `contract_id`: globally unique URI identifier, immutable
- `contract_meta`: authorship, lifecycle, and labeling metadata
- `agent`: identity and classification block; agent types: `ai_assistant`, `planning_agent`, `task_agent`, `data_agent`, `infrastructure_agent`, `autonomous_system`, `composite_agent`, `human_node`
- `principal`: authorizing party block; principal types: `human`, `agent`, `organization`, `system`; `delegation_depth` field for chain tracking
- `authority.data_access`: data domain permissions with classification ceiling, operations, and conditions
- `authority.tools`: tool/API permissions with permitted actions, rate limits, and conditions
- `authority.decisions`: autonomous decision type definitions with thresholds and conditions
- `authority.delegation`: sub-agent delegation permissions with depth limiting
- `constraints`: contract-level prohibitions with override semantics; evaluated before permissions
- `escalation`: human-in-the-loop rules with trigger, target, timeout, and priority
- `connectivity`: connectivity mode model with four states (connected, degraded, disconnected, contested); `pre_authorization_envelope` field (working draft); reconnection behavior
- `binding`: enforcement mode (native, overseer, infrastructure); lifecycle behavior (on_violation, on_expiry, immutable, audit_required)
- Shared `$defs`: `condition` (contextual permission conditions), `constraint` (prohibition definition)

**Documentation**
- `docs/concepts.md`: full primitive definitions, intellectual lineage, consumption patterns, domain applicability
- `README.md`: spec overview, primitives table, examples, contributing guide, versioning

**Examples**
- `examples/enterprise-procurement-agent.json`: enterprise AI governance pattern; connected operation; native enforcement mode
- `examples/critical-infrastructure-grid-agent.json`: OT/ICS disconnected operations pattern; infrastructure enforcement mode; pre-authorization envelope (working draft)

**Governance**
- `CONTRIBUTING.md`: RFC process, contribution types, issue labels, domain contributor guide
- `LICENSE`: Apache 2.0

### Working Draft Fields

The following fields have defined schema and semantics but normative evaluation requirements are in progress. Community contributions are invited.

- `connectivity.pre_authorization_envelope`: cryptographic signing requirements and state reconciliation protocol
- `connectivity.reconnection_behavior.await_ratification`: ratification evaluation semantics

### Open Profiles (RFC Invited)

- `disconnected-operations`: defense/C2, OT/ICS
- `critical-infrastructure`: power, pipeline, water, transportation
- `defense-national-security`: autonomous systems in denied/contested environments
- `autonomous-field-operations`: field robotics, remote operations
- `high-latency-autonomous`: space-based autonomous systems
