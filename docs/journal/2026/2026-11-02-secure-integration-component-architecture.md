# Textus EAI: secure integration component and artifact harness

Date: 2026-11-02
Status: architecture direction / proposed scope (not implementation claim)

## Core decisions
- Public-facing Textus EAI is a conventional API; Driver/Adapter/Provider implementations may use an **internal DSL** aligned with CNCF UoW for controlled effects.
- **Security Complete CAR** can execute directly in CNCF runtime subject to admission and runtime policy. Other CARs and non-CAR programs run in a **Docker sandbox by default**. Policy-controlled relaxation is possible, explicit, audited and bounded by hard limits.
- Docker isolates arbitrary Python, CLI, Java/Node and other tools while keeping implementation free. All interactions must cross controlled input/output/network/log boundaries; Docker alone is not proof of this property. Disable direct network by default, Docker socket, broad host mounts, privileged mode, host IPC and unmediated credentials; apply OS limits.
- **Application Produces, EAI Manages**: integration code produces output in a private workspace and writes stdout/stderr; EAI classifies, validates, places, retains, shares and removes artifacts and logs according to policy. No application-specific storage paths or cleanup logic.
- **Private by Default, Explicit Publication**: controlled, time-limited sharing when needed; permanent public release via distinct admission. Provider capabilities vary (S3 presigned URLs, GCS signed URLs, Drive sharing/session semantics).
- Performance and reliability harness: bounded streaming/chunking, backpressure, quotas, timeouts, cancellation, retry/idempotency, orphan reclamation, evidence/trace. Arbitrary tool algorithms are not inherently optimized.

## Integration Component contract
Identity/version; typed input/output; owner/tenant/classification/provenance; capabilities; resource budget; retention and destination policy; execution profile; results/logs/trace. CAR and Docker adapters present a consistent public API. Future distribution via CNCF Component Repository is a **deferred option**, not initial scope.

## Runtime flow
EAI API -> contract/admission -> execution selector -> (Security Complete CAR + UoW | Docker sandbox) -> output admission -> private artifact store / authorized target; logs and temporary resources handled centrally. Sandbox output is untrusted until validated. Runtime failure leaves evidence and recoverable orphan cleanup.

## Trust caveats
Only managed I/O is governed; enforce network/volume/IPC boundaries and prevent bypass. Signed images and admission do not establish arbitrary code safety. Distinguish physical placement from logical retention and data owner authority. Policy relaxation must be visible to Control Center/TRACE.

## Initial acceptance slices
1. Mail attachment Excel -> private artifact -> TKL Material ingestion; source attribution, malware-safe handling, cleanup.
2. Mobile media direct upload control: TKL -> EAI -> Drive session, client -> Drive data; verify upload before TKL admission, support relay fallback.
3. Confidential CSV may not be published by a tool; orphan and failed-execution cleanup; audit policy overrides.

## Development follow-up
Review existing TEAI phase 1 and contract notes before changing phases. Reuse CNCF UoW, Security Complete, CAR admission and sandbox primitives rather than duplicating them. Reference CNCF security incident assessment and TKL ingestion journal.
