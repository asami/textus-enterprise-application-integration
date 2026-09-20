# Knowledge Lake as an OpenClaw + TEAI Reference Integration

Date: 2026-09-21

## Decision

Use the Google Workspace Knowledge Lake / TKL flow as a representative TEAI use case for agent-mediated physical integration.

TEAI owns the external integration contract and enterprise semantics. OpenClaw can perform the concrete Google Workspace/tool interaction. TKL receives only a provider-neutral KnowledgeCandidate/event contract.

```text
Google Workspace / NotebookLM
  -> OpenClaw: provider/tool-specific physical access
  -> TEAI: enterprise integration semantics
  -> TKL: semantic knowledge ingestion
  -> Textus World
```

This complements the existing FTP/SFTP deterministic endpoint scenario and the OpenClaw Continuation scenario. Together they exercise direct deterministic integration, agent-mediated SaaS integration, and goal delegation/continuation.

OpenClaw must not become the owner of Integration Binding, CNCF Job/Workflow association, correlation, delivery policy, audit/provenance, or the Textus-facing contract. Those remain TEAI responsibilities.
