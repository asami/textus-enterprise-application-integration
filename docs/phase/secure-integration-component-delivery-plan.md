# Secure Integration Component delivery plan

status=planned
planned_at=2026-11-02
owner=textus-enterprise-application-integration
references=../journal/2026-11-02-secure-integration-component-architecture.md

## Scope and dependencies
Follow Phase 1's existing enterprise event/continuation contract; do not change Phase 1 closure criteria. Reuse CNCF Security Complete CAR admission, UoW, security profiles and sandbox execution where actually implemented. Missing primitives are explicit CNCF dependencies, not shadow implementations in EAI. A normal public API serves applications; constrained internal DSL serves managed Driver/Adapter/Provider implementations. No new workflow engine, storage platform or repository distribution service.

| Slice | Deliverable | Status | Acceptance |
| --- | --- | --- | --- |
| EAI-S01 | Freeze Integration Component input/output, identity, owner/tenant, execution policy and artifact contract; bind CNCF APIs and identify missing upstream primitives. | OPEN | Typed contract and rejected invalid/bypass requests. |
| EAI-S02 | Execution selection: Security Complete CAR may run in-process subject to admission; all others Docker by default; audited explicit relaxation with hard limits. | OPEN | Selection/rejection tests; no CAR label-only trust. |
| EAI-S03 | Docker runner: private input RO, output-only mount, restricted network/IPC, no Docker socket, no broad host mounts, non-root, resource/time limits and trace. | OPEN | Negative network/mount/privilege tests; crash cleanup; inspect host-specific enforcement. |
| EAI-S04 | Artifact/log lifecycle: private staging, output validation, policy-selected destination, retention, temporary cleanup, orphan reconciliation and idempotent completion. | OPEN | Confidential CSV never written to public store; failed runs do not publish; logs protected. |
| EAI-S05 | Mail attachment Excel -> TKL Material ingestion via adapter; provenance, owner resolution and duplicate/retry handling. | OPEN | Original retained, no direct public staging, Material registration verified. |
| EAI-S06 | TKL-controlled mobile upload preparation, Drive resumable session adapter, direct transfer completion verification, relay fallback. | OPEN | Client->Drive data path, verified registration, wrong file/size/owner rejected, restart/expiry handled. |
| EAI-S07 | Controlled temporary sharing and publication admission using provider-specific capabilities; performance bounds and connected acceptance. | OPEN | TTL/revocation semantics stated by provider; no implicit public sharing. |

## Sequence
S01 -> S02/S03 -> S04 -> S05 and S06 -> S07. Deliver S05/S06 as vertical slices with their TKL owners. Do not gate the existing TKL direct upload on completion of all generic EAI functions; agree a compatible transition and fallback. Component Repository distribution is future scope only.

## Acceptance boundary
Security defaults must be active in a clean installation, not only configurable. Verify OS/container enforcement rather than assuming Docker alone closes every channel. Distinguish signed/qualified CAR from arbitrary safe code. Keep artifact owner authority in TKL, and treat upload receipt as provisional until TKL confirms admission. Record actual implementation, provider limitations and test evidence before closing any item.
