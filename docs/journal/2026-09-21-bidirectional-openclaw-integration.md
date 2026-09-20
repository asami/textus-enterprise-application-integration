# Bidirectional OpenClaw Integration

Date: 2026-09-21

## Refinement

The earlier OpenClaw integration discussion emphasized external systems entering TEAI through OpenClaw. This is incomplete: CNCF/TEAI also needs to actively use OpenClaw capabilities during Workflow execution.

OpenClaw is therefore modeled as a **bidirectional Intelligent Integration Edge**.

## Interaction model

Four patterns are distinguished:

- EVENT: OpenClaw -> TEAI
- INVOCATION: TEAI -> OpenClaw
- DELEGATION: TEAI -> OpenClaw
- CONTINUATION: OpenClaw -> TEAI

The important distinction is between INVOCATION and DELEGATION.

INVOCATION requests a concrete known capability and remains orchestration even when OpenClaw performs the physical access.

DELEGATION gives OpenClaw responsibility for achieving a goal. It belongs to continuation semantics because the external participant may decide which tools, channels, AI processing, and interaction sequence are needed.

## CNCF alignment

This reinforces the need to preserve ORCHESTRATION / CONTINUATION binding semantics in CNCF.

OpenClaw usage must not automatically mean CONTINUATION. The semantic boundary depends on whether Textus is invoking a known capability or delegating responsibility for a goal.

## Knowledge Lake impact

The Knowledge Lake scenario uses both directions. OpenClaw may deliver a Knowledge Candidate/event into TEAI, but a TKL Workflow may later ask OpenClaw to retrieve a concrete Workspace resource or delegate a broader investigation of related materials.

This makes the Knowledge Lake scenario a useful test of all four TEAI/OpenClaw interaction patterns.
