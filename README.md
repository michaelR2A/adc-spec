# Agent Delegation Contract (ADC)

**Open specification for AI agent authorization — define what agents may do, decide, and escalate.**

`v0.1 Working Draft` | Apache 2.0 | R2 Advisory LLC

---

## What ADC Is

An **Agent Delegation Contract** is a formal, machine-readable contract that defines the authorized scope of an AI agent's actions:

- What **data** it may access, at what classification level, with what operations
- What **decisions** it may make autonomously, under what conditions and thresholds
- What **tools** it may invoke, with what permitted actions
- What **escalation conditions** route decisions to a human or higher authority
- What **authority** the agent retains when it cannot reach its evaluation runtime

ADC is a **specification**, not an implementation. Any runtime may evaluate ADCs. The language is open. Runtimes are built on top of it.

---

## The Problem ADC Solves

AI agents are making consequential decisions inside enterprise systems, critical infrastructure, and autonomous platforms today. They access data, approve transactions, trigger workflows, control physical systems, and spawn other agents. The authorization model governing those decisions is, in most deployments, absent.

Existing security infrastructure answers different questions:

| Tool | Question It Answers |
|---|---|
| IAM | Who is this human, and what resources may they access? |
| DLP | Is this data leaving the perimeter? |
| DSPM | What sensitive data exists and where? |
| SIEM | What happened, after the fact? |

None of them answer: **what is this agent authorized to decide, right now, in this context?**

ADC answers that question. It provides a contract language for expressing agent authorization that is formal enough to be machine-evaluated, human-readable enough to be auditable, and general enough to apply from enterprise SaaS to disconnected field operations.

---

## Intellectual Lineage

ADC formalizes concepts that **C2 (Command and Control) doctrine** has applied to human-operated systems for decades, now expressed as machine-readable contracts for autonomous agents.

| C2 Concept | ADC Equivalent |
|---|---|
| Delegation of Authority | ADC delegation chain: authority constrained downward at every level |
| Rules of Engagement | ADC constraint set: explicit prohibitions that override permissions |
| Commander's Intent | Root intent contract: principal's objective governing the delegation chain |
| PACE Communications | Connectivity model: connected, degraded, disconnected, contested |
| Pre-Delegated Authority | Pre-authorization envelope: decisions cleared before contact is lost |
| Positive / Procedural Control | Connected vs. disconnected operation modes |

This is not analogy. The problems are structurally identical at different layers of abstraction. ADC brings machine-readable precision to authority models C2 doctrine has long handled through human interpretation.

---

## Scope

ADC is designed for any environment where autonomous agents make consequential decisions. The specification defines a **domain-invariant core** with **normative profiles** for specific operational contexts.

| Domain | Profile Status |
|---|---|
| Enterprise AI Governance | Normative (v0.1) |
| Critical Infrastructure (OT/ICS) | Working Draft — contributors needed |
| Defense / National Security | Working Draft — contributors needed |
| Autonomous Field Operations | Placeholder — RFC open |
| High-Latency Autonomous (Space) | Placeholder — RFC open |

---

## Repository Structure

```
spec/
  adc-v0.1.json          # JSON Schema — the normative specification
examples/
  enterprise-procurement-agent.json    # Enterprise AI governance pattern
  critical-infrastructure-grid-agent.json  # Disconnected/OT operations pattern
docs/
  concepts.md            # Primitives, design rationale, consumption patterns
  field-reference.md     # Field-by-field reference (forthcoming)
  authoring-guide.md     # How to write an ADC (forthcoming)
  profiles/              # Domain-specific normative profiles (forthcoming)
  rfcs/                  # RFC archive
CONTRIBUTING.md          # How to contribute
CHANGELOG.md             # Version history
LICENSE                  # Apache 2.0
```

---

## Core Primitives

| Primitive | Definition |
|---|---|
| **Agent** | An entity with decision authority: capacity to evaluate context and take consequential action |
| **Principal** | The authorizing party accountable for the agent's authorized actions |
| **Authority** | The complete enumerated scope of what an agent is permitted to do. Default: deny. |
| **Constraint** | An explicit prohibition that overrides permissions. Evaluated before authority. |
| **Delegation** | Transfer of a bounded authority subset to a sub-agent. Can only be constrained downward, never amplified. |
| **Escalation** | Condition under which the agent suspends autonomous action and routes to a human or higher authority |
| **Binding** | Enforcement mode and lifecycle semantics for the contract |
| **Connectivity Mode** | The agent's relationship to its evaluation runtime: connected, degraded, disconnected, contested |
| **Pre-Authorization Envelope** | Decisions cleared for autonomous execution when the agent cannot reach its runtime |

Full definitions: [`docs/concepts.md`](docs/concepts.md)

---

## Minimal Example

```json
{
  "adc_version": "0.1",
  "contract_id": "urn:adc:example:procurement-agent:2026-04-16:v1",
  "agent": {
    "agent_id": "procurement-agent-001",
    "agent_type": "task_agent"
  },
  "principal": {
    "principal_id": "cfo@example.com",
    "principal_type": "human"
  },
  "authority": {
    "decisions": [
      {
        "decision_type": "approve_purchase_order",
        "autonomous": true,
        "thresholds": [
          { "field": "purchase_order.amount_usd", "operator": "lte", "value": 10000 },
          { "field": "purchase_order.vendor_id", "operator": "in", "value": ["{{finance.approved_vendor_list}}"] }
        ]
      }
    ]
  },
  "escalation": [
    {
      "trigger": "purchase_order.amount_usd > 10000",
      "escalate_to": "cfo@example.com",
      "timeout": { "seconds": 86400, "on_timeout": "deny" },
      "priority": "routine"
    }
  ],
  "binding": {
    "enforcement_mode": "native",
    "on_violation": "deny_and_escalate",
    "audit_required": true
  }
}
```

Full examples: [`examples/`](examples/)

---

## Enforcement Modes

ADC defines three enforcement modes. The contract format is identical across all three. What differs is where evaluation happens and whether the agent participates.

**Pattern 1 — Native Consumption**
The agent reads its ADC, self-governs, and calls the evaluation runtime to validate decisions. The agent is ADC-aware.

**Pattern 2 — Overseer Validation**
Sub-agents execute without ADC awareness. An overseer agent validates outputs against the governing ADC before propagation. Bridge pattern for non-ADC-native agents.

**Pattern 3 — Infrastructure Enforcement**
The evaluation runtime intercepts agent actions at the infrastructure boundary. Enforcement operates regardless of agent awareness.

---

## Connectivity Model

ADC specifies agent authority behavior under four connectivity states. Enterprise deployments operating in fully connected environments may omit the `connectivity` block entirely. Disconnected and contested profiles are required for field, OT/ICS, and defense deployments.

| Mode | Authority Behavior |
|---|---|
| `connected` | Full contract authority, real-time evaluation |
| `degraded` | Pre-authorized subset, escalations queued |
| `disconnected` | Pre-authorization envelope governs; deny on ambiguity |
| `contested` | Minimal safe actions only; halt and report on reconnect |

---

## Relationship to Existing Standards

ADC is complementary to, not a replacement for:

- **NIST AI RMF**: ADC provides the machine-readable contract layer for operationalizing AI RMF governance controls at the agent level
- **EU AI Act**: ADC contract terms can express and enforce AI Act obligations (human oversight, prohibited practices) at the agent authorization layer
- **ISO 42001**: ADC supports AI management system controls by formalizing agent authorization as auditable policy
- **OpenAPI / AsyncAPI**: ADC governs what an agent is authorized to do with APIs it can invoke; it does not describe the APIs themselves
- **SAML / OAuth**: ADC extends identity and access concepts to agent-level decision authorization; it assumes identity is established by existing IAM

---

## Contributing

ADC needs practitioners from enterprise AI governance, defense/national security, critical infrastructure, and autonomous systems. The spec is stronger with domain expertise from each community.

**Where contributions are most needed right now:**

- `disconnected-operations` profile: defense/C2 practitioners and OT/ICS engineers
- `critical-infrastructure` profile: power, pipeline, water, transportation operators
- Example ADCs: any domain, any agent pattern
- Field reference documentation

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the RFC process, issue labels, and how to get started.

---

## Versioning

| Version | Status | Date |
|---|---|---|
| v0.1 | Working Draft | April 2026 |
| v1.0 | Target: Q4 2026 | — |

v0.1 is a working draft. Fields marked `WORKING DRAFT` in the schema have defined semantics but normative evaluation requirements are in progress. Schema changes are possible before v1.0. v1.0 will be the first locked release.

Breaking changes to v0.1 follow the RFC process. See [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## License

Apache 2.0. See [`LICENSE`](LICENSE).

The ADC specification is open. Runtimes built on ADC may be open or proprietary.

---

## Maintainer

**R2 Advisory LLC**
Michael Ruiz, Founder and Managing Director
[r2advisory.com](https://r2advisory.com)

R2 Advisory maintains the ADC specification and the reference enforcement runtime, [Arbiter](https://r2advisory.com/arbiter). ADC is designed to be evaluable by any conformant runtime. Arbiter is never a requirement for ADC adoption.
