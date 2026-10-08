# Control Center AI Integration Strategy Alignment

Date: 2026-10-08
Status: consumer alignment
Authority: textus-control-center/docs/strategy/ai-integrated-development-operations.md

TEAI aligns with the Control Center target architecture that runs Dot and OpenClaw as complementary capabilities.

## OpenClaw operating role

OpenClaw is primarily the operational/EAI agent. Its default economic deployment should prefer a local LLM where the required capability is sufficient, especially for continuous scheduling, collection/classification, routine external-service handling and overnight work.

TEAI should not use OpenClaw as a routine wrapper around Codex programming. When an EAI/Slack-originated request requires software engineering, route the semantic request to Control Center/sm-workflow; sm-workflow selects/binds the Codex execution context.

OpenClaw may still replace Dot as a Semantic Management Agent when needed, but that management role remains distinct from Codex implementation.

TEAI retains deterministic integration/workflow authority. Agent memory, prompts and sessions are not canonical integration state.
