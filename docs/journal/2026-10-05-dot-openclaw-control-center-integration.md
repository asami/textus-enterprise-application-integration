# Dot / OpenClaw / Control Center Integration

Date: 2026-10-05
Status: architectural direction

## Decision

TEAI keeps OpenClaw as an operational/EAI agent while allowing Textus Control Center to use a higher-level semantic management agent such as OpenAI Dot.

The preferred deployment may therefore be:

```text
Dot
  -> Textus Control Center
       -> sm-workflow / development
       -> TEAI
            -> OpenClaw
            -> external systems
```

This is a deployment preference, not a hard dependency.

When Dot is unavailable or undesirable, OpenClaw may also implement the semantic-management role:

```text
OpenClaw
  -> Textus Control Center
       -> sm-workflow
       -> TEAI / external systems
```

## TEAI responsibility

TEAI remains responsible for deterministic integration orchestration, durable job/workflow state, external-system events, and bounded delegation to AI agents.

OpenClaw is especially suitable for operational/EAI work requiring external applications, AI-mediated interaction, or non-deterministic handling. Deterministic operations remain in TEAI/CNCF workflows rather than being moved into the agent.

Dot is not introduced as a TEAI authority or required runtime dependency.

## Agent independence

TEAI contracts should express integration goals, events, continuations, results, and evidence without embedding Dot/OpenClaw-specific sessions, prompts, model names, or UI concepts.

Agent-specific details belong to adapters/providers. This permits Dot, OpenClaw, ZeroClaw, or future agents to be selected according to deployment and operational requirements.

## Control Center and human interaction

Textus Control Center provides the integrated management view across development and operations. Slack is the primary near-term human interaction channel for notifications, approvals, and instructions.

Where human authority is required, the authoritative decision is recorded through the relevant typed application/workflow operation. Slack messages and agent memory remain interaction context, not canonical state.

Service-bus/event integration should be preferred over agent polling so that agents are activated by meaningful operational changes.
