# Profile: Enterprise AI Governance

**Status:** Normative (v0.1)
**Domain:** Enterprise AI agents operating in connected organizational environments
**Applies to:** Financial services, healthcare, technology, professional services, government

---

## Scope

This profile defines normative ADC usage for AI agents deployed in enterprise environments with reliable connectivity to an evaluation runtime. It covers the common patterns for authorization governance across LLM assistants, task agents, planning agents, and data agents operating against enterprise systems.

---

## Connectivity Assumption

Enterprise AI governance deployments assume `connectivity.assumed_mode: connected`. The `connectivity` block may be omitted entirely. Real-time evaluation by the authoritative runtime is assumed for all decisions.

---

## Required Fields

The following fields are required for all contracts conforming to this profile, beyond the core schema requirements:

| Field | Requirement | Rationale |
|---|---|---|
| `contract_meta.expires_at` | Required | Stale permissions accumulate risk. All enterprise contracts must have explicit expiry. |
| `contract_meta.authored_by` | Required | Accountability chain requires a named author. |
| `binding.audit_required` | Must be `true` | Enterprise compliance and incident response depend on complete audit trails. |
| `binding.immutable` | Must be `true` | Contract mutability undermines audit integrity. Changes require new contract IDs. |
| `escalation` | At least one rule required | All enterprise agents must have at least one escalation condition defined. |

---

## Classification Levels

The following classification ceiling mapping is recommended for enterprise deployments:

| Level | Typical Scope |
|---|---|
| `public` | Marketing materials, published documentation |
| `internal` | General business information not for external distribution |
| `confidential` | Client data, financial records, HR records, strategic plans |
| `restricted` | Regulated data (PII, PHI, PCI), board materials, M&A information |

Agents should be granted the lowest classification ceiling sufficient for their function.

---

## Recommended Constraint Patterns

All enterprise AI governance ADCs should include constraints addressing the following:

**Data Exfiltration**
```json
{
  "id": "no-export-above-internal",
  "description": "Agent may not export data classified above internal to any external destination.",
  "applies_to": "data_access",
  "on_violation": "deny_and_halt"
}
```

**External Communication Scope**
```json
{
  "id": "restrict-external-comms",
  "description": "Agent may not send communications to recipients outside the organization's approved domains.",
  "applies_to": "tool_invocation",
  "on_violation": "deny_and_escalate"
}
```

**Sub-Agent Delegation Control**
```json
{
  "id": "no-unauthorized-delegation",
  "description": "Agent may not spawn sub-agents or issue ADCs unless explicitly permitted in authority.delegation.",
  "applies_to": "delegation",
  "on_violation": "deny_and_halt"
}
```

---

## Escalation Patterns

### Threshold Escalation
Route to a human when a decision exceeds the autonomous threshold:
```json
{
  "trigger": "decision_type == 'approve_purchase_order' AND purchase_order.amount_usd > 10000",
  "escalate_to": "cfo@yourorg.com",
  "timeout": { "seconds": 86400, "on_timeout": "deny" },
  "priority": "routine"
}
```

### Constraint Violation Escalation
All enterprise ADCs should route constraint violations to a security contact:
```json
{
  "trigger": "constraint_violation",
  "escalate_to": "security-ops@yourorg.com",
  "timeout": { "seconds": 3600, "on_timeout": "deny_and_halt" },
  "priority": "immediate"
}
```

### New Entity Escalation
Route decisions involving new or unvetted entities to a human:
```json
{
  "trigger": "entity.status == 'unvetted'",
  "escalate_to": "risk-team@yourorg.com",
  "timeout": { "seconds": 172800, "on_timeout": "deny" },
  "priority": "routine"
}
```

---

## Regulatory Alignment

ADC contracts conforming to this profile provide the authorization contract layer for the following frameworks:

| Framework | ADC Alignment |
|---|---|
| NIST AI RMF | `authority` + `constraints` implement GOVERN and MANAGE controls at the agent level; `binding.audit_required` supports MEASURE |
| EU AI Act (human oversight obligations) | `escalation` conditions implement mandatory human oversight for high-risk AI decisions |
| ISO 42001 | ADC contracts are the auditable policy artifacts for AI management system controls |
| OWASP GenAI Data Security 2026 | `authority.data_access` + `constraints` implement data access controls addressing OWASP agent data security risks |

---

## Example

See `examples/enterprise-procurement-agent.json` for a complete normative example conforming to this profile.
