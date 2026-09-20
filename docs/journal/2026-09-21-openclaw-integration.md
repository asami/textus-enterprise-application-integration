# OpenClaw as the Intelligent Integration Edge

Date: 2026-09-21

## Decision

TEAI will integrate with OpenClaw as an important reference Delegation Endpoint, but will not attempt to duplicate OpenClaw's broad external connector/tool ecosystem.

The motivation is both architectural and practical. Reimplementing Slack, Gmail, GitHub, Web/SaaS, AI providers, and conversational/human channels inside TEAI would create a large connector-maintenance burden while adding little Textus-specific value.

## Resulting split

TEAI directly handles integrations whose behavior is naturally deterministic and enterprise-oriented, such as FTP/SFTP file arrival, message brokers, database/file integration, and deterministic service invocation.

OpenClaw handles agent-mediated interaction with external services, AI/agent/tool execution, and conversational/human channels.

TEAI/CNCF remain authoritative for enterprise semantics and execution: Integration Binding, Job, Workflow, StateMachine, policy, continuation identity, audit/provenance, and deterministic processing.

## Two endpoint families

The discussion led to a useful conceptual distinction:

1. **Deterministic Endpoint** — TEAI/CNCF knows the concrete operation to perform.
2. **Delegation Endpoint** — TEAI delegates a goal to an external participant and waits for a continuation result.

OpenClaw is the initial reference for the second family.

## Why this matters

This changes the role of an EAI platform. A traditional EAI system typically needs a concrete connector and integration flow for each external service. TEAI can instead delegate some external interaction at the level of enterprise intent.

For example, a workflow can request "obtain approval from the responsible person" rather than encoding "post this Slack message, wait for this callback, parse this reply". OpenClaw may choose Slack, email, AI/tool interaction, or another supported mechanism. The enterprise workflow receives the normalized result through the Continuation Protocol.

## Constraint

This does not mean all external integration should pass through OpenClaw.

Direct TEAI integration remains appropriate where deterministic behavior, throughput, latency, strict delivery semantics, governance, or operational simplicity require it. OpenClaw delegation is primarily for goal-oriented, agent-mediated, conversational, tool-rich, or otherwise non-deterministic interaction.

## Consequence for TEAI implementation

TEAI should prioritize:

- a small set of core deterministic enterprise endpoints;
- a provider-neutral Delegation Endpoint SPI;
- CNCF Continuation Protocol integration;
- correlation, policy, security, audit/provenance and result normalization.

It should **not** prioritize building a large catalog of SaaS connectors.

This keeps TEAI focused on enterprise integration semantics while allowing OpenClaw to serve as the intelligent integration edge.
