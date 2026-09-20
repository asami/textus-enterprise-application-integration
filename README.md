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

- deterministic enterprise endpoints and adapters where direct integration is appropriate: FTP/SFTP, messaging, file arrival, database, deterministic HTTP/REST, etc.
- delegation endpoints for agent-mediated integration, with OpenClaw as the initial reference
- enterprise events
- integration bindings and trigger policies
- external/canonical DTO and message mapping
- transformation and routing
- delivery concerns such as deduplication and idempotency
- integration audit/provenance
- inbound event-to-workflow and outbound workflow-to-endpoint integration

TEAI should also avoid becoming a connector museum. SaaS, conversational channels, human interaction, and AI/tool ecosystems that can be mediated effectively by an agent gateway should normally be delegated to that gateway rather than reimplemented as TEAI-specific connectors.

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

## Connector strategy

TEAI distinguishes **deterministic endpoints** from **delegation endpoints**.

```text
TEAI Endpoint
  +-- Deterministic Endpoint
  |     FTP / SFTP / MQ / File / DB / deterministic REST / ...
  |
  +-- Delegation Endpoint
        OpenClaw
          +-- Slack / chat
          +-- Gmail / mail services
          +-- GitHub
          +-- Web / SaaS tools
          +-- AI / agents
          +-- Human interaction
          +-- other tool ecosystems
```

The default architectural rule is: **do not duplicate an external-service connector in TEAI merely because TEAI could implement it**. If OpenClaw can safely mediate a non-deterministic or agent-oriented interaction, TEAI should express the enterprise goal and delegate it. TEAI retains the enterprise semantics: event/binding, workflow association, policy, audit/provenance, continuation identity, and execution guarantees.

For example, a CNCF Workflow may delegate the goal "obtain approval from the responsible person" without knowing whether OpenClaw uses Slack, email, another collaboration service, or additional AI interaction. The continuation result returns the enterprise-level outcome to the workflow.

This keeps provider-specific interaction at the edge while preserving deterministic enterprise control in TEAI/CNCF.

## Physical integration responsibility

Physical integration with external systems is a TEAI responsibility. TEAI supports two implementation paths:

```text
External System
  |
  +-- deterministic integration --> TEAI Endpoint
  |
  +-- agent/tool-mediated SaaS --> OpenClaw --> TEAI
```

OpenClaw is therefore an integration-edge implementation used by TEAI, not a replacement for TEAI's enterprise integration semantics. OpenClaw can absorb provider-specific APIs, authentication/tooling details, conversational interaction and SaaS connector differences. TEAI retains endpoint/event semantics, Integration Binding, policy, correlation, delivery guarantees, audit/provenance, Job/Workflow association and Continuation Protocol integration.

### Knowledge Lake reference integration

The Google Workspace Knowledge Lake is a representative use case:

```text
Google Workspace / NotebookLM
          |
       OpenClaw
  physical/tool access
          |
         TEAI
  integration boundary
          |
         TKL
 KnowledgeCandidate ingestion
          |
     Textus World
```

TKL is intentionally isolated from Google-specific physical access. TEAI presents a provider-neutral Knowledge Candidate/event contract to TKL while OpenClaw handles suitable Google Workspace/SaaS interaction on the external edge.

## Bidirectional OpenClaw integration

OpenClaw is a **bidirectional Intelligent Integration Edge** for TEAI, not only an ingress connector.

```text
                    OpenClaw
                Intelligent Edge
                /              \
           ingress            egress
              |                  ^
              +------ TEAI ------+
                       |
                      CNCF
                       |
                    Workflow
```

TEAI/OpenClaw interaction has four primary patterns:

1. **EVENT** — OpenClaw -> TEAI. An event or externally observed condition enters the enterprise integration runtime.
2. **INVOCATION** — TEAI -> OpenClaw. TEAI requests a concrete external tool/service operation whose intent and operation are already determined by the Textus side.
3. **DELEGATION** — TEAI -> OpenClaw. TEAI delegates a goal and allows OpenClaw/agents/tools to determine how to achieve it.
4. **CONTINUATION** — OpenClaw -> TEAI. A delegated activity completes/fails/cancels and returns the normalized result needed to continue the CNCF Workflow.

INVOCATION and DELEGATION are intentionally distinct. Invocation is orchestration of a known external capability; delegation transfers responsibility for achieving a goal across a continuation boundary.

Example:

```text
INVOCATION:
  "Retrieve file X from Google Drive"

DELEGATION:
  "Find relevant project material for this KnowledgeCandidate
   and return grounded supplementary information"
```

This distinction should be reflected in TEAI contracts and mapped consistently to CNCF ORCHESTRATION / CONTINUATION semantics.
