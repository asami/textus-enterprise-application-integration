# TEAI Phase 1 contract planning

Date: 2026-10-02
Status: planning record; product implementation and acceptance pending

## Request and preparation

The user requested specification consideration and a Phase 1 development plan,
starting from the architecture journal and concrete CNCF Job/Workflow/
Continuation alignment.

Initialized the existing ai/directive submodule at the recorded gitlink
`e25b94e42d3aa58a0af87d42049dae7a8463a26c`. The root AGENT.md, AGENTS.md and
RULE.md symlinks now resolve. No directive version upgrade was selected.

## Investigation and proposed scope

Read the TEAI README, OpenClaw notes and all four architecture/integration
journals, then inspected CNCF source, codecs and executable-specification source.
The inspected CNCF HEAD was `f7cf5bc04c11b9f74e09d61b8199452c5edb3275`;
its Job area also contains concurrent local work. No upstream code was edited
or tested by this planning task.

The proposal reuses typed WorkflowProtocolV1, Continuation SPI and declared
completion Operations. It separates managed Job completion from business
Workflow completion, and CNCF duplicate rejection from successful result replay.
It also distinguishes the event/entity WorkflowEngine route from the typed
durable WorkflowInstance route; their concrete managed entry binding remains
an early implementation proof item.

The first planned proof uses one FileArrived fixture, a deterministic read
invocation, one delegated document-summary goal, and a separate-turn returned
result resuming the same Workflow. CNCF execution is real; the external edge
is a deterministic protocol test driver. Live OpenClaw/provider integration,
distributed delivery and comprehensive crash recovery remain later work and
are not claimed by the initial proof.

The architecture journal's logical authority / physical orchestration split
and Program > Local LLM > Frontier AI preference are preserved. No model is
needed merely to test the integration contract.

## Outputs

- [Specification discussion and CNCF evidence](../notes/minimum-teai-integration-contract.md)
- [Phase 1 checklist, sequence and acceptance examples](../phase/phase-1.md)

This record does not close Phase 1, promote the proposal into an implemented
contract, run a demonstration, or record any commit/publication.
