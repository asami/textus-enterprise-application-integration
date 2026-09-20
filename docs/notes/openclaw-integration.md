# OpenClaw Integration

## Purpose

OpenClaw is the initial reference **Delegation Endpoint** for TEAI. It provides an intelligent integration edge for external services, conversational channels, AI/agent execution, tools, and human interaction.

TEAI must not depend on OpenClaw-specific concepts in its core integration model. The integration is expressed through provider-neutral continuation and endpoint contracts.

## Architectural boundary

```text
External world
  Slack / Gmail / GitHub / Web / SaaS / Human / AI
                         |
                         v
                     OpenClaw
                Intelligent Edge
                         |
                 Continuation Protocol
                         |
                         v
                       TEAI
       Integration Binding / Policy / Mapping
                         |
                         v
                       CNCF
          Job / Workflow / Operation / State
```

### OpenClaw responsibilities

OpenClaw should normally absorb external-service-specific concerns for agent-mediated integration:

- channel/tool/provider adapters;
- service-specific APIs and interaction differences;
- conversational interaction;
- AI/agent execution and tool use;
- human interaction through collaboration channels;
- selection and coordination of external tools needed to achieve a delegated goal.

### TEAI responsibilities

TEAI retains enterprise-integration semantics:

- deterministic endpoints where direct integration is appropriate;
- enterprise event normalization;
- integration binding and trigger policy;
- canonical DTO/message mapping where required;
- delegation endpoint abstraction;
- continuation correlation and identity;
- integration policy and security boundary;
- audit/provenance;
- delivery guarantees appropriate to the integration;
- association with CNCF Job/Workflow execution.

### CNCF responsibilities

CNCF owns runtime execution semantics:

- Job;
- Workflow;
- StateMachine;
- Operation;
- JudgmentAction;
- suspension/continuation semantics;
- execution state and monitoring.

## Endpoint categories

```text
IntegrationEndpoint
  +-- DeterministicEndpoint
  |     +-- Ftp/Sftp
  |     +-- MessageQueue
  |     +-- File
  |     +-- Database
  |     +-- DeterministicHttp
  |
  +-- DelegationEndpoint
        +-- OpenClaw
        +-- future agent gateway
        +-- human-task provider
```

This is a semantic distinction, not necessarily a concrete class hierarchy.

## Connector strategy

TEAI should not reproduce every SaaS connector supported by OpenClaw.

A useful default:

| Integration | Preferred owner |
| --- | --- |
| FTP/SFTP/file arrival | TEAI |
| MQ/event broker | TEAI |
| database integration | TEAI |
| deterministic REST/API integration | TEAI/CNCF |
| Slack/chat interaction | OpenClaw |
| Gmail/mail interaction requiring agent behavior | OpenClaw |
| GitHub interaction requiring agent/tool behavior | OpenClaw |
| AI/LLM/agent execution | OpenClaw |
| human conversational interaction | OpenClaw |
| general SaaS/tool interaction driven by a goal | OpenClaw |

This is a default, not an absolute rule. A deterministic, high-volume, strongly governed integration may justify a direct TEAI endpoint even when OpenClaw has a connector.

## Goal-oriented delegation

The workflow should preferably delegate an enterprise goal rather than a sequence of provider-specific tool calls.

Example:

```text
CNCF Workflow
  -> TEAI DelegationEndpoint
  -> Goal: ObtainApproval(applicationId, responsibleRole)
  -> OpenClaw
       -> choose interaction channel
       -> contact responsible person
       -> clarify if necessary
       -> obtain response
  -> ContinuationResult: Approved / Rejected / ...
  -> resume CNCF Workflow
```

The workflow therefore does not need to know whether Slack, email, or another service was used.

## Continuation contract direction

The contract should carry at least:

- continuation/workflow correlation identity;
- delegated goal/capability;
- input payload/reference;
- expected result contract;
- policy/context required for execution;
- provenance/audit metadata;
- completion/failure/cancellation result.

Detailed DTO design should be aligned with the CNCF Continuation Protocol rather than invented independently in TEAI.

## Design rule

**OpenClaw handles how to interact with the external world; TEAI/CNCF retain what the enterprise process means and how its execution is governed.**
