# Textus EAI secure integration — development proposal

Date: 2026-11-02
Status: proposal in journal, NOT a formal Phase

## Scope and boundary
EAI exposes conventional public APIs; managed Driver/Adapter/Provider may use an internal DSL mapped to CNCF UoW. CNCF owns the generic External Component execution selector, Security Complete CAR qualification, Docker sandbox and resource/trace primitives. EAI must not duplicate that runtime. Existing TEAI Phase 1 event-to-continuation proof stays unchanged.

## Proposed delivery slices
1. Define Integration Component contract atop CNCF external execution: identity, input/output, data owner/classification/provenance, destination, retention and execution requirements. Component Repository distribution is future scope.
2. Implement EAI artifact management: private staging, output validation, logical-to-physical placement, temporary cleanup, orphan recovery, protected logs, idempotent delivery and retention policy. Application produces; EAI manages.
3. Provider/adapter boundaries: mail/attachment, Google Drive, optional S3/GCS; private by default; explicit publication admission and provider-specific temporary sharing. External tools operate in CNCF-managed sandbox and cannot bypass EAI destinations.
4. Mail Excel -> Knowledge Lake Material vertical slice: preserve original, resolve owner via trusted mapping, attach provenance, handle duplicate/unknown delivery and sandbox parsing/scanning.
5. Mobile direct-upload vertical slice: TKL control -> EAI Drive preparation/session -> client uploads bytes directly to Drive -> EAI verifies -> TKL admits; retain TKL relay fallback and existing digest/reservation semantics.
6. Operational acceptance: bounded streaming/chunking, retry/timeout, private artifact and log policy, no public-storage misplacement, unauthorized access and crash cleanup. Separate provider-supported guarantees from post-upload validation.

## Dependencies and sequencing
First freeze CNCF External Component execution interface and TKL Material handoff; implement artifact lifecycle and one concrete provider, then the two vertical slices. EAI's initial mail and mobile slices need not wait for future Component Repository distribution or broad multi-provider support. Maintain Security Complete direct execution vs Docker-default rule in CNCF, not a second EAI certification.

## Phase migration
Review TEAI phase convention and all planned phases before mapping these slices into formal numbered phases. Provide goal, stage/step, executable acceptance, dependencies, closure evidence; do not treat this journal as a Phase or assert completion. Existing Phase 1 remains unchanged.
