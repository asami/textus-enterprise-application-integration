# Phase 1 planning and related-document review

Date: 2026-10-02 (Asia/Tokyo)
Review ID: TEAI-P1-PLAN-REVIEW-20261002
Reviewer: Codex, owning review chat `01a0e3d2-a450-7641-8bad-a769f102658a`, agent `/root`
Disposition: FINDINGS — two contract-planning gaps to resolve in P1-03/P1-04
Scope: planning review, not implementation acceptance or formal Phase closure

## Reviewed state and authority

The user requested a Phase 1 review, close examination of related documents,
and a journal record of the result. Only this review record is edited.

The local branch was clean at `9c962f9a6600b0cfd4b5cf72988e05eb89afa9ba`.
`git fetch origin` located the requested plan in
`402f99d51f14bf8e4784e1dd3fcac59b7439c2ee`
(`WIP(checkpoint): Preserve TEAI Phase 1 contract planning`). Review reads used
that exact commit through `git show`; the local branch and working files were
not advanced to it. A checkpoint is not implementation acceptance.

Primary reviewed documents, with line numbers below referring to that commit:

- [Phase 1](https://github.com/asami/textus-enterprise-application-integration/blob/402f99d51f14bf8e4784e1dd3fcac59b7439c2ee/docs/phase/phase-1.md)
- [Minimum contract proposal](https://github.com/asami/textus-enterprise-application-integration/blob/402f99d51f14bf8e4784e1dd3fcac59b7439c2ee/docs/notes/minimum-teai-integration-contract.md)
- README, including its new Current development section.
- `docs/journal/2026-10-02-phase-1-contract-planning.md`.

Related inputs: both OpenClaw notes; all four 2026-09-21 TEAI journals;
root AGENTS/RULE and the document-lifecycle and Phase/checklist rules at the
recorded directive gitlink; TKL README, Phase 1 and relevant architecture/model
sections. No repository-local rules or agent guide were present in TEAI.

CNCF was resolved by repository identity: local
`/Users/asami/src/dev2025/cloud-native-component-framework` has origin
`asami/goldenport-cncf`. Its HEAD was
`6cbe877c53c116639feb11fa80546e79c051804c` with ongoing working changes.
Read-only inspection covered JobEngine/managed Command documentation,
WorkflowEngine, JclRuntimeBridge, WorkflowProtocolV1, start/result codecs,
ContinuationRuntime, ContinuationSpiAdapter, WorkflowCompletionServiceOperation,
WorkflowInstanceAtomicTransitionV1, and relevant protocol-spec source.
This is corroboration against the available working sources, not verification
of the proposal's exact upstream snapshot. The proposal's CNCF commit
`f7cf5bc04c11b9f74e09d61b8199452c5edb3275` was not in this local object database;
its accompanying uncommitted Job changes were not available as a frozen input.
No upstream work was edited or accepted by this review.

## Current Boundary Blockers

These are gaps in the proposed contract/acceptance coverage, not observed
implementation vulnerabilities. They block treating the corresponding contract
as frozen; they do not prevent continued Stage 1 investigation.

### CB-TEAI-P1-001 [P2] Define the completion caller's trust boundary

Evidence: minimum contract lines 98–101 explicitly bind the authenticated
source for EVENT, but lines 113–121 describe CONTINUATION admission only through
payload and correlation/revision/snapshot/evidence checks. Phase 1 AC-06
(line 94) covers malformed/stale/duplicate results, not an unauthorized caller
submitting an otherwise matching result. The driver is described as deterministic,
but deterministic behavior does not define who may call completion or inspection.

The available CNCF `admitResultC` validates matching references and declared
facts; it does not authenticate the transport caller. Private claim ownership
is recovered internally. Keeping the claim token private therefore does not,
by itself, establish that the submitter is the delegated participant.
This review does not assert that CNCF's wider ingress security is absent.

Risk: an implementation could meet every listed correlation example while
leaving caller admission unspecified, or mistake knowledge of a WorkOrder for
authorization to complete it.

Required clarification in P1-03/P1-04: choose and document either an explicitly
trusted, isolated test-driver boundary or the existing CNCF ingress subject /
operation policy that binds a caller to the allowed endpoint/work. State the
corresponding lookup visibility. If exposed beyond the trusted driver, add an
acceptance case where an unauthorized caller presents otherwise valid result
fields and cannot resume or alter the workflow; an authorized submission must
still work afterward. Reuse existing framework security. Do not invent a TEAI
authentication service, new secret claim field or OpenClaw authentication scheme.

Owner: TEAI. Fix boundary: Phase 1 contract and acceptance examples only.

### CB-TEAI-P1-002 [P2] Make lost-start-response reconciliation testable

Evidence: minimum contract lines 154–156 and 173 require lookup after uncertain
start or lost response; Phase 1 AC-08 (line 96) says to query using the same
identity. The correlation chain contains several distinct identities, while
the concrete public query remains undecided in minimum contract line 211.

Risk: when the *first start response* is lost, the caller may have neither
Job ID nor WorkflowHandle. A query that requires either cannot satisfy that
case. An empty receipt lookup also cannot by itself prove that no Job/Workflow
started. This ambiguity can lead to a second start or leave the stated
acceptance example untestable, even with one runtime and sequential requests.
It does not require a crash or a distributed exactly-once guarantee.

Required clarification in P1-03/P1-04: specify lookup using an identity already
known before submission, such as the authenticated source/event key; define
what is recorded before/after start and how absent, pending, known-started and
unconfirmed observations differ. Explicitly forbid interpreting missing
association evidence as permission to launch again. Split AC-08 into lost
start-response and lost completion-response cases, with deliberate response
loss and observable Job/Workflow counts. Bind the needed lookup to an existing
CNCF/TEAI public route. If the selected CNCF version cannot support it, retain
the gate as open and record the upstream dependency; do not add a substitute
execution/recovery engine.

Owner: TEAI. Fix boundary: the already planned receipt/query contract and
timeout acceptance examples, within the existing single-runtime failure model.

## Hygiene Ledger Candidates

### HYG-TEAI-P1-001 [P3] Make CNCF source references portable and reproducible

Minimum contract lines 68–80 use thirteen `../../../goldenport-cncf/...`
links. This workstation uses a different checkout basename and parent path;
those links do not resolve here. GitHub-relative traversal is not a portable
cross-repository source citation either. The instruction to resolve repository
identity is helpful but does not make the hyperlinks usable.

Use repository-qualified paths plus commit-permalink citations for committed
evidence. At the existing P1-04 dependency-binding step, distinguish committed
upstream evidence from observations of dirty files and record the exact usable
artifact/ABI selection. The current mixed HEAD/working-copy observation must
not become proof of a published dependency. This is documentation maintenance;
the plan already leaves dependency adoption and focused proof open.

Owner/boundary: TEAI contract proposal's source-evidence section. No upstream
edit, source hash protocol, dependency upgrade or plan rebaseline is requested.

## Development Candidate Ledger Entries

None newly introduced by this review. Live OpenClaw/FTP/SFTP/Google Workspace,
multi-worker/crash guarantees, and a single enterprise Job spanning the entire
business workflow are already explicitly deferred. They are not Phase 1
blockers. TKL's real KnowledgeCandidate flow is a separate acceptance boundary;
the fixed document-summary fixture cannot be cited as completion of TKL Phase 1.

## Positive findings and existing gates

- Ownership is consistent: TEAI integrates; CNCF owns execution, claims and
  transitions; the Edge owns physical work within delegated scope.
- The proposal preserves all four interaction meanings without confusing them
  with the four CNCF continuation variants. The initial WORK_ORDER to TERMINAL
  subset is explicit; unsupported driver variants stop rather than being guessed.
- The Event/entity WorkflowEngine route is not passed off as typed durable
  Start. Managed Job success and terminal business outcome are separated.
- Post-commit issuance, separate-call completion, stable WorkflowHandle,
  duplicate rejection versus successful replay, and failed/cancelled business
  outcomes are distinguished. Hash-derived event identities are not introduced.
- The atomic capability is correctly treated as a provider obligation rather
  than proof of a transaction. No production exactly-once claim is made.
- Program-first execution and provider neutrality agree with the architecture
  journals. A deterministic driver is appropriate for this contract proof.
- Status is honest: only P1-01/P1-02 are checked; P1-03 through P1-12 remain
  open. Missing design/spec, dependency coordinates, generated ABI and focused
  proof are explicit upcoming work, not falsely claimed completed deliverables.
- P1-04 must prove the concrete managed start/completion route before dependent
  implementation. Full evidence must cover real CNCF and generated ABI, not
  only mocked Job/Workflow calls. This is an existing gate, not a new feature.

## Validation and review boundary

- Documentation-only change set: four files, 384 added lines at the reviewed
  checkpoint. All project documentation in that tree was read; pertinent CNCF
  and TKL sources were cross-checked as described above.
- Target Programs: none. TEAI has no product source, executable specs, build.sbt,
  project.yaml or CML in the reviewed tree. Scala naming checks are not applicable.
- Executable Specification Check: AC-01 through AC-09 are prospective GWT
  examples; no executable acceptance exists to run or assess as passed.
- CAR lint: not applicable to this documentation-only, pre-bootstrap repository.
  P1-12 already schedules it for the implemented CAR. CBD integration is not
  configured in this tree; no CBD MCP or invented integration task was called.
- No SBT, Cozy, runtime, live provider or network behavior tests were run.
  Git fetch only retrieved review inputs. No commit, merge, push or Phase-state
  transition is performed by this journal entry.
- Review-record whitespace validation: `git diff --check` plus an untracked-file
  no-index whitespace check. This does not establish product correctness.

## Review disposition draft

- Source: `TEAI-P1-PLAN-REVIEW-20261002`, reviewer `/root` in the chat above.
- Kind: standalone Phase-planning/document review; workflow: review-only plus
  explicitly authorized journal recording; affected identity: TEAI Phase 1.
- Frozen target: TEAI commit `402f99d51f14bf8e4784e1dd3fcac59b7439c2ee`.
- Current Boundary Blockers: `CB-TEAI-P1-001`, `CB-TEAI-P1-002`.
- Hygiene: `HYG-TEAI-P1-001`; Development Candidates: empty.
- Journal-ready finding text and ownership: the complete corresponding entries
  above, persisted in this file at the user's request.
- Authority boundary: TEAI planning/contract documents; read-only CNCF/TKL
  corroboration; no architecture expansion, upstream edits, implementation
  acceptance, new authentication/recovery engine, or live-provider acceptance.
- No Phase full-review ledger or control transition is claimed. The Phase owner
  must admit any later formal review using its then-current records and evidence.
