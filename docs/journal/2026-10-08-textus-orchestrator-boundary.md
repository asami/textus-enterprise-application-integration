# Textus Orchestrator Boundary

Date: 2026-10-08
Status: architecture alignment
Authority: textus-orchestrator architecture

TEAI remains the enterprise/application integration subsystem. Cross-participant semantic routing is assigned to Textus Orchestrator rather than TEAI or OpenClaw.

Textus Orchestrator may route ENTERPRISE_INTEGRATION work to TEAI. TEAI then owns deterministic integration/workflow behavior and may bind OpenClaw/local LLM for bounded operational/EAI work.

OpenClaw is not the normal programming intermediary. SOFTWARE_ENGINEERING routes to sm-workflow, which binds Codex.

Dot/Astra and OpenClaw/local LLM may both run concurrently as Orchestrator participants/providers. TEAI does not need to know or own the global routing policy.
