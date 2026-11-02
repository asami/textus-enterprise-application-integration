# Phase 2: Secure EAI Artifact and Provider Integration

status=planned
planned_at=2026-11-02
owner=textus-enterprise-application-integration
checklist=inline EAI2-01..06 below

## Goal and source

Deliver one private mail-Excel-to-Material route, then mobile direct upload,
using CNCF-managed execution and TKL-owned Material admission. Application
produces; EAI manages artifacts, transfers, provider operations and retention.

Sources: [development proposal](https://github.com/asami/textus-enterprise-application-integration/blob/d71aa7a2e6a15f69a2f2f6a34fad810e9c481af4/docs/journal/2026/2026-11-02-secure-eai-development-proposal.md)
and [architecture direction](https://github.com/asami/textus-enterprise-application-integration/blob/d71aa7a2e6a15f69a2f2f6a34fad810e9c481af4/docs/journal/2026/2026-11-02-secure-integration-component-architecture.md).
[Phase 1](phase-1.md) remains unchanged, including its original event/continuation
proof and unfinished items. This plan adds no obligations to that acceptance.

## Ownership and dependency gates

- [CNCF Phase 118](https://github.com/asami/goldenport-cncf/blob/main/docs/phase/phase-118.md)
  owns execution selection, qualification, isolation, budgets and lifecycle.
  A qualified bounded Docker handoff is sufficient for the first EAI tool route;
  direct-CAR qualification or whole immutable rotation need not finish first.
- [TKL Phase 2](https://github.com/asami/textus-knowledge-lake/blob/main/docs/phase/phase-2.md) owns logical
  identity, owner/tenant authorization, journal and Material admission. Align
  prepare/status/finalize/registration and private-delivery evidence before effects.
- EAI owns conventional public APIs and managed provider/adapter effects.
  Reuse CNCF UoW, process and continuation contracts; no EAI workflow engine,
  second Security Complete certification or general raw-command public API.
- Inventory actual Phase 1 capabilities at EAI2-01. Reuse delivered contracts;
  if the selected route needs an unfinished Phase 1 capability, name that item
  as a prerequisite rather than declaring Phase 1 complete or duplicating it.

## Implementation order

1. EAI2-01 aligns contracts with CNCF/TKL and selects one real mail source and
   Google Drive as the initial storage provider. Select trusted recipient/source
   to owner mapping with the operator; sender text is not owner authority.
2. EAI2-02 and TKL2-02 build one complete route using private artifacts and the
   original Excel workbook. Add the executable specifications in the same batch.
   Use a deterministic attachment fixture first; live mail/Drive qualification
   remains required and is reported separately.
3. EAI2-03 closes private/public destination, log, duplicate and crash branches
   around that working route and provides the reusable artifact/Drive handoff.
4. EAI2-04/05 and TKL2-03/04 connect mobile control and direct bytes, then failure,
   restart and relay behavior. Core owns reusable transfer; Framework/App compose it.
5. EAI2-06 qualifies both joined routes and hands off deployment/operation evidence.

Provider-specific restrictions must be documented before promising an upload
constraint. Drive resumable sessions are sensitive capabilities, not S3/GCS
presigned URLs. Distinguish restrictions enforced by the provider from checks
possible only after upload. Unsupported mandatory controls reject or use an
explicitly admitted fallback; they do not silently weaken the contract.

## Stage: Secure integration delivery

Stage Status:
- Current status: OPEN
- Owner: EAI
- Update rule: EAI2-01..06 and acceptance EA-01..07 are the closure basis; attach actual evidence before checking an item.

- [ ] EAI2-01: Promote Integration Component and EAI/TKL contracts into design/spec:
  identity/version, typed inputs/results, owner/tenant/classification/provenance,
  logical destination, retention, execution requirement and correlation. Record
  actual CNCF/Phase 1 capabilities and the initial mail/Drive deployment choices.
- [ ] EAI2-02: One mail attachment traverses private staging -> constrained
  scan/parse -> TKL registration/admission with private persistence acknowledged.
  Preserve the original workbook and message/attachment/source identifiers.
  A registration request may precede delivery; admission must await its evidence.
- [ ] EAI2-03: Central artifact/log lifecycle supports output validation, logical
  placement, private defaults, explicit publication admission, provider-specific
  temporary sharing, retention, retry/idempotency and recoverable orphan cleanup.
  Tools cannot bypass EAI destinations or write confidential output publicly.
- [ ] EAI2-04: Real Shared Drive preparation/status/finalize creates a restricted
  new-file resumable upload session for TKL; mobile bytes go directly to Drive.
  No server credential is exposed, and session URIs are protected/redacted.
- [ ] EAI2-05: Verify provider file identity, reservation, size, expected digest
  and private placement before returning admissible delivery evidence. Cover
  ambiguous result, duplicate, expiry, destination change, cancellation, orphan
  cleanup and TKL relay fallback with equivalent authorization/audit.
- [ ] EAI2-06: Required executable checks, real provider acceptance, bounded
  large-transfer proof and integrated independent review pass. Deliver reusable
  provider contract/configuration, tested artifact and operating limitations.

## Executable acceptance

| ID | Given / When | Then |
| --- | --- | --- |
| EA-01 | Trusted mailbox mapping receives an Excel attachment | Original and provenance survive into one private TKL Material; sandbox parsing has no publication authority |
| EA-02 | Sender spoofing, unknown mapping or wrong-tenant request | No owner is inferred from sender text and no unauthorized Material is admitted |
| EA-03 | Tool emits confidential CSV or secrets in output/logs | Unauthorized public placement/sharing is denied; logs/evidence remain bounded and protected |
| EA-04 | Same delivery is repeated or completion is unknown | Query/reconcile the existing delivery; no duplicate business registration or blind resubmission |
| EA-05 | Android/iPhone uploads multi-GB media through prepared Drive session | Bytes bypass TKL/EAI servers; bounded client/provider transfer and independently verified completion |
| EA-06 | File/digest/destination differs, session expires or service crashes | No success inference; protected evidence and owner-scoped recovery/cleanup, preserving admitted originals |
| EA-07 | Direct transfer cannot be used and relay is explicitly selected | Same owner/reservation/integrity/admission contract; fallback is visible, not a hidden default |

Shared Drive is the first live target; preserve the same contract for My Drive
shared folders and report its live qualification separately. Temporary sharing
never implies permission for permanent public publication.

## Closure and deferred scope

Use one integrated review per connected delivery and focused closure for repairs;
do not repeat whole suites for helpers. Reuse producer evidence at its actual
scope and retain required repository release checks. Mail and mobile can be
handed off separately; Phase closure requires the entire checklist.

Component Repository distribution, S3/GCS implementations, all mail systems,
hostile-code VM isolation and a custom operations dashboard are deferred. TRACE
must expose the required correlation and policy facts without waiting for a new UI.

Workbench consumes TKL query/provenance, not EAI transport APIs. See its
[consumer acceptance plan](https://github.com/asami/textus-knowledge-workbench/blob/main/docs/phase/eai-material-consumer-plan.md).

## Small-business security baseline allocation — 2026-11-02

Apply the [Textus standard/configurable/external contract](https://github.com/asami/goldenport-cncf/blob/main/docs/spec/small-business-security-baseline-and-options.md)
to EAI2-01/03/04/06.

- [ ] SB-01..03: private artifacts, scoped provider credentials, intended
  destinations and diagnostic redaction hold by default after necessary provider
  onboarding. Supported sharing/processing choices remain consistent with TKL;
  no UI-only restriction or silent widening of provider authority.
- [ ] Handoff states actual supported policy, operator update/recovery duties
  and optional IdP/HSM/offline prerequisites. Do not present absent external
  controls as active or require them for the normal supported route.

Reuse connected acceptance and producer evidence. This does not add every
optional strengthening integration to EAI Phase 2.
