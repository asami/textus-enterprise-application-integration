# TEAI Architecture and Goals

Date: 2026-09-21

## Context

TEAI is being introduced as a new Textus component for Enterprise Application Integration.

The starting point is a conventional EAI requirement: enterprise events such as an FTP/SFTP file arrival trigger application processing. In Textus, the triggered processing should not be implemented by a TEAI-specific workflow engine. TEAI bridges the event to CNCF, starts a Job/Workflow, and relies on CNCF Job and Workflow management for execution state and observability.

The design is extended for AI-era enterprise applications by making goal delegation and continuation a first-class integration mechanism.

## Architectural goals

1. Provide a common integration model for traditional enterprise endpoints and AI/agent environments.
2. Convert enterprise events into CNCF execution requests through explicit integration bindings.
3. Use CNCF Job management to manage enterprise execution units and CNCF Workflow to manage process progression.
4. Avoid duplicating Workflow, StateMachine, Job, or execution-monitoring functionality in TEAI.
5. Support both inbound event-triggered processing and outbound endpoint invocation.
6. Preserve ORCHESTRATION and CONTINUATION as distinct binding semantics.
7. Allow a CNCF Workflow to delegate goal-oriented processing to AI/agent environments through the Continuation Protocol.
8. Keep the TEAI core independent of OpenClaw; OpenClaw is a reference continuation endpoint/provider.
9. Support progressive formalization: exploratory AI processing can evolve into policies, workflows, and deterministic operations.
10. Maintain traceability, audit/provenance, idempotency and other enterprise-integration guarantees across the boundary.

## Reference architecture

```text
FTP/SFTP ---- FileArrived ----+
Webhook ----- Request --------+
MQ ---------- Message --------+--> TEAI --> CNCF Job --> CNCF Workflow
OpenClaw ---- Event ----------+                         |
                                                        +--> Operation
                                                        +--> Endpoint
                                                        +--> Continuation
                                                               |
                                                               +--> OpenClaw / AI / Human
```

TEAI is the integration layer; CNCF is the execution runtime.

## Job and Workflow

A Job represents the enterprise execution unit: what was accepted, from where, when it started, its current/result status, and the Workflow execution associated with it.

A Workflow represents progression of the processing itself.

For an inbound file:

```text
FileArrived
  -> Integration Binding
  -> start Job
  -> start Workflow
  -> validate / transform / invoke / complete
```

This separation allows CNCF Dashboard/Job/Workflow facilities to provide a unified operational view rather than introducing a TEAI-specific execution console.

## Continuation as the differentiator

Traditional EAI primarily orchestrates known operations and endpoints. TEAI additionally allows a workflow to delegate a goal to an external participant and resume when that participant returns a result.

This is not merely asynchronous RPC. The delegated participant may perform non-deterministic, goal-oriented work.

```text
Workflow
  -> delegate Goal
  -> Continuation Protocol
  -> AI/Agent participant
  -> continuation result
  -> resume Workflow
```

OpenClaw is the initial reference environment for this capability.

The protocol and model must nevertheless be provider-neutral so that other AI agents, agent gateways, and human-task implementations can participate.

## Progressive formalization

An important operational goal is to allow enterprise work to begin with AI assistance before its stable process is fully understood.

Repeated operation can expose stable transformations, decisions, and sequences. Those parts can then migrate from AI delegation toward JudgmentAction/policy, Workflow, and ultimately deterministic Operation implementations.

TEAI therefore supports a continuum between exploratory integration and formally guaranteed enterprise processing rather than forcing all integration logic to be specified up front.

## Initial scope direction

The first implementation should validate two complementary scenarios:

- traditional EAI: FTP/SFTP file arrival -> event -> binding -> CNCF Job/Workflow;
- AI-native EAI: CNCF Workflow -> Continuation Protocol -> OpenClaw -> result -> workflow continuation.

Detailed protocol DTOs, endpoint SPI, delivery guarantees, and CNCF binding contracts should be derived from these scenarios rather than designing a large generic EAI framework first.

## Continuation IoC: logical authority and physical orchestration

OpenClaw integration uses the Continuation Protocol as an inversion-of-control boundary.

The authority split is intentional:

- **TEAI / Textus is the logical control authority.** It owns integration meaning, workflow/job state, routing policy, admission decisions, auditability, and the semantic decision about what should happen next.
- **OpenClaw is the physical orchestration authority for work delegated to it.** It owns the active execution loop: invoking TEAI/Textus, receiving a Continuation, executing the requested external/AI work, returning results, and advancing through subsequent Continuations.
- OpenClaw is therefore not merely a passive adapter, but it is also not the owner of the business/workflow semantics.

The normal AI execution shape is:

```text
TEAI / Textus
 logical authority
       ^
       | Continuation Protocol
       v
OpenClaw
 physical orchestration
   |            |
   v            v
 Codex      Local LLM
             (Ollama)
```

OpenClaw may select and operate Codex or a local LLM within the execution responsibility delegated through the protocol. TEAI may also gain direct Codex or local-LLM routes where useful, but those optional routes do not change the primary authority split.

This supersedes interpretations in which OpenClaw is the top-level logical routing authority or in which TEAI directly drives every physical worker invocation. Continuation Protocol deliberately allows TEAI/Textus to retain semantic authority while yielding physical control back to OpenClaw between decisions.

## Cost-aware Continuation Dispatcher

AI execution cost is a first-class architectural concern for Continuation-based orchestration.

A Continuation response often does not itself require a frontier model. OpenClaw may need only to interpret the returned request/result, identify the appropriate execution/thinking logic, and invoke that logic. A small local model is therefore a candidate **Continuation Dispatcher**.

The dispatcher is part of OpenClaw's physical orchestration layer, not TEAI/Textus logical authority.

```text
TEAI / Textus
  logical authority
       |
       | Continuation response
       v
OpenClaw
  physical orchestration
       |
       v
Local dispatcher model
       |
       +-- deterministic operation
       +-- lightweight local judgment -> local LLM
       +-- implementation/review/high reasoning -> Codex
       +-- external integration -> OpenClaw integration logic
       +-- human approval -> yield
       +-- next protocol call -> TEAI/Textus
```

The local dispatcher should normally solve a bounded classification/selection problem rather than perform the expensive work itself. Inputs should expose explicit state, requested goal/action, ReasoningLevel/Presentation, allowed execution mechanisms, and relevant observations. Output should be schema-constrained, for example an executor/logic/reasoning selection.

Routing must combine deterministic constraints with model judgment. The dispatcher must not invent protocol transitions or acquire workflow authority. Candidate selections remain subject to validation/admission, and ambiguous, unsupported, high-risk, or high-reasoning work can escalate to Codex.

The initial Mac mini experiment shows Qwen 3.5 4B through native Ollama is viable enough to evaluate for this dispatcher role. This is a hypothesis to test with real Continuation fixtures, not a permanent model choice.

The cost objective is to avoid spending Codex/frontier-model capacity merely to decide *which* deterministic operation, local reasoning routine, Codex task, integration action, or human continuation should run next. Frontier-model calls should be reserved for work whose semantic difficulty warrants them.

Future evaluation should compare local-dispatch and Codex decisions over representative Continuation fixtures, including correctness, escalation behavior, latency, and AI cost.
