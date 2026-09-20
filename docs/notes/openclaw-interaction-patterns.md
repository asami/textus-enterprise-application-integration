# OpenClaw Interaction Patterns

## Purpose

TEAI uses OpenClaw as a bidirectional Intelligent Integration Edge. The relationship is not limited to OpenClaw delivering external events into TEAI; TEAI can actively use OpenClaw capabilities from CNCF Workflow execution.

## Four interaction patterns

### EVENT

Direction: `OpenClaw -> TEAI`

An external event, observation, or tool-mediated notification is normalized into the TEAI integration model and may trigger a CNCF Job/Workflow.

Examples:

- a SaaS-side event observed through an OpenClaw connector;
- availability of a Knowledge Candidate in a Google Workspace Knowledge Lake;
- a human response received through a collaboration channel.

### INVOCATION

Direction: `TEAI -> OpenClaw`

TEAI asks OpenClaw to perform a concrete external capability. The operation is known and controlled by the Textus/CNCF side.

Examples:

- retrieve a specific Google Drive resource;
- post a specified message to a collaboration channel;
- obtain a known external resource through an available tool.

Invocation belongs primarily to ORCHESTRATION semantics even when the physical adapter happens to be OpenClaw.

### DELEGATION

Direction: `TEAI -> OpenClaw`

TEAI delegates a goal rather than a fixed external operation. OpenClaw may select tools, channels, agents, AI models, or interaction sequences needed to achieve the goal.

Examples:

- obtain approval from the responsible person;
- investigate related project materials and return grounded supplementary information;
- collect information needed to resolve a Knowledge Candidate.

Delegation crosses a CONTINUATION boundary. The external participant temporarily owns how the goal is achieved.

### CONTINUATION

Direction: `OpenClaw -> TEAI`

OpenClaw returns the result of delegated work so that the associated CNCF Workflow can continue.

The result should carry correlation/continuation identity and a normalized completion state such as completed, failed, or cancelled, plus the contract-defined result/provenance.

## Relationship to CNCF

```text
CNCF Workflow
   |
   +-- ORCHESTRATION
   |      |
   |      +-- TEAI deterministic endpoint
   |      +-- TEAI -> OpenClaw INVOCATION
   |
   +-- CONTINUATION
          |
          +-- TEAI -> OpenClaw DELEGATION
          +-- OpenClaw -> TEAI CONTINUATION
```

The physical use of OpenClaw does not itself imply continuation. A concrete OpenClaw tool invocation can remain ordinary orchestration.

## Knowledge Lake example

Both directions are useful in the TKL reference architecture.

Ingress:

```text
Google Workspace / NotebookLM
  -> OpenClaw
  -> EVENT
  -> TEAI
  -> TKL ingestion workflow
```

Workflow-driven external use:

```text
TKL Workflow
  -> TEAI
  -> DELEGATION: investigate related project material
  -> OpenClaw
       -> Drive / Gmail / Web / AI / ...
  -> CONTINUATION with grounded result
  -> TKL Workflow resumes
```

A deterministic lookup can instead use INVOCATION:

```text
TKL Workflow
  -> TEAI
  -> INVOCATION: retrieve resource X
  -> OpenClaw
  -> result
```

## Contract direction

The TEAI/OpenClaw boundary should avoid one undifferentiated request type. Contracts should make the interaction intent explicit enough to distinguish at least EVENT, INVOCATION, DELEGATION and CONTINUATION.

Detailed DTOs must be aligned with CNCF Workflow/Continuation contracts and should not be independently invented in TEAI.
