# Textus Enterprise Application Integration (TEAI)

TEAI is the enterprise application integration layer for Textus.

It connects enterprise events and endpoints to CNCF Operations and Workflows, while allowing workflow steps to delegate non-deterministic work to AI/agent environments through the Continuation Protocol.

## Core idea

TEAI treats integration as a bridge between external enterprise endpoints and the CNCF execution model.

Typical inbound flow:

```text
Enterprise Endpoint
  -> Enterprise Event
  -> TEAI Integration Binding
  -> CNCF Job
  -> CNCF Workflow
  -> Operations / external endpoints
```

For example, an FTP/SFTP file arrival can raise a `FileArrived` event. TEAI resolves the corresponding integration binding, starts a CNCF Job, and starts the bound Workflow. Job management provides the execution-management view of the workflow run.

## Responsibilities

TEAI owns integration concerns such as:

- endpoints and adapters: FTP/SFTP, HTTP/Webhook, messaging, mail, agent gateways, etc.
- enterprise events
- integration bindings and trigger policies
- external/canonical DTO and message mapping
- transformation and routing
- delivery concerns such as deduplication and idempotency
- integration audit/provenance
- inbound event-to-workflow and outbound workflow-to-endpoint integration

TEAI does **not** introduce another workflow or job engine. Workflow, StateMachine, Job, JudgmentAction, execution monitoring, and related runtime semantics belong to CNCF.

## AI/Agent integration

A defining TEAI capability is delegation from an enterprise workflow to an AI/agent environment.

```text
CNCF Workflow
     |
     +-- deterministic Action
     |
     +-- AI/Agent delegation
             |
             +-- Continuation Protocol --> OpenClaw / other agent environment
                                             |
                                      goal-oriented processing
                                             |
             <-- continuation result --------+
     |
     +-- workflow continues
```

This is intentionally different from merely invoking an AI API. The workflow can delegate a goal to an external participant, suspend at a continuation boundary, and continue when the participant returns its result.

OpenClaw is an important reference integration, but it is not part of TEAI's core model. TEAI must remain independent of a particular agent gateway.

## Orchestration and continuation

TEAI preserves the distinction between two integration styles:

- **ORCHESTRATION** — CNCF controls the processing sequence and invokes deterministic operations/endpoints.
- **CONTINUATION** — work is delegated to an external participant such as an AI agent or human; CNCF continues the workflow when the result returns.

This makes AI/human goal delegation a first-class enterprise-integration mechanism rather than an ad-hoc external call.

## Progressive formalization

TEAI supports a development and operation pattern in which work may begin as exploratory AI-driven processing and become deterministic as the organization learns the stable process:

```text
AI / Agent delegation
        |
        v
repeatable judgment / policy
        |
        v
CNCF Workflow
        |
        v
deterministic Operation
```

The boundary between AI-managed and programmatically guaranteed processing can therefore move over time without changing the overall enterprise integration architecture.

## Initial reference scenarios

1. **FTP/SFTP file arrival**
   - detect file arrival
   - create an enterprise event
   - resolve an integration binding
   - start a CNCF Job and Workflow
   - manage and observe execution through CNCF Job/Workflow facilities

2. **OpenClaw continuation**
   - reach an AI/agent delegation point in a CNCF Workflow
   - delegate the goal through the Continuation Protocol
   - allow OpenClaw to perform AI/agent processing
   - receive the continuation result
   - resume the same workflow instance

These two scenarios deliberately cover both traditional EAI and AI-native enterprise integration.
