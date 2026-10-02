# Phase 1 — Minimum integration contract and one continuation workflow

Status: open
Updated: 2026-10-02
Owner: textus-enterprise-application-integration

## Purpose

Enterprise Event を CNCF の管理された実行へ接続し、外部 goal 委譲の返却結果で
同じ WorkflowInstance を再開できる最小 TEAI を成立させる。

## Reading and authority

- Direction: [Architecture journal](../journal/2026-09-21-teai-architecture-and-goals.md)
- Specification discussion: [Minimum TEAI contract](../notes/minimum-teai-integration-contract.md)
- Interaction vocabulary: [OpenClaw patterns](../notes/openclaw-interaction-patterns.md)
- Planning history: [2026-10-02 journal](../journal/2026-10-02-phase-1-contract-planning.md)

仕様案は notes に置き、Stage 1 で design/spec に確定する。この Phase は実装順序と
checklist を管理する。notes の記載だけで実装・テスト成功を主張しない。

## Boundary

- TEAI が所有: Event、Binding、型付き mapping、外部 interaction adapter、受領対応、相関と integration audit。
- CNCF が所有: Job、WorkflowInstance、Continuation、claim、revision、UoW、実行状態と再開。
- Edge が所有: 委譲された範囲内の物理実行。初期実証では決定的な契約テスト用ドライバ。
- 変更対象: TEAI repository。CNCF/Cozy は依存契約の読取と利用。必要な上流変更は owning repository の別作業として扱う。

## Scope and acceptance profile

単一 runtime、逐次配送、一つの固定 Binding、一つの文書処理 Workflow。
四つの interaction を一つの実証に含め、CNCF の公開 Operation/生成 ABI を通す。
同じ event の再送、結果の重複、不正な相関、goal failure/cancellation を小さな
fixture で確認する。起動 Job の完了と Workflow の業務結果を区別する。

実 OpenClaw 接続、LLM 品質、FTP/SFTP/Google Workspace adapter、multi-worker、
crash/restart を含む end-to-end exactly-once、汎用再送基盤、運用 dashboard は範囲外。
この Phase の成功は実 CNCF とテスト用 Edge 間の契約実証であり、実 OpenClaw や
本番配送保証の受入ではない。

## Stage 1 — Contract and dependency binding

Stage Status:
- Current status: IN_PROGRESS
- Owner: TEAI
- Update rule: 下記 checklist と参照証拠を更新する。全項目完了が closure basis。

- [x] P1-01: 固定 gitlink `e25b94e42d3aa58a0af87d42049dae7a8463a26c` の ai/directive を初期化し、root directive links が読める。
- [x] P1-02: architecture と CNCF の現行 source/spec の対応・差異を仕様検討案に記録した。ソース読取であり実行検証ではない。
- [ ] P1-03: Event/Binding、四つの interaction、相関、逐次重複、failure model を design/spec に確定し、fixture の型を決めた。
- [ ] P1-04: 使用する CNCF/Cozy の版、生成 ABI、managed Command → typed Start → completion の公開経路を特定し、Job と Workflow の対応を focused proof で確認した。

## Stage 2 — Component foundation and event admission

Stage Status:
- Current status: OPEN
- Owner: TEAI
- Update rule: 下記 checklist と検証結果を更新する。全項目完了が closure basis。

- [ ] P1-05: 現行 Cozy/CML による最小 component、生成/build、公開 Operation、focused executable specification の実行が成立する。
- [ ] P1-06: 固定 Event/Binding fixture の受領が実 CNCF Job と一つの typed Workflow を開始し、source/event → binding/version → Job/Workflow の対応を照会できる。
- [ ] P1-07: 不正入力・Binding 不一致/曖昧性を拒否し、同じ event の逐次再送では既存対応を返す。異なる入力で同一キーを使った場合は上書きしない。

## Stage 3 — Delegation and same-instance resume

Stage Status:
- Current status: OPEN
- Owner: TEAI
- Update rule: 下記 checklist と検証結果を更新する。全項目完了が closure basis。

- [ ] P1-08: 決定的な文書取得 INVOCATION を ORCHESTRATION で実行し、goal DELEGATION を CNCF WORK_ORDER として返す。
- [ ] P1-09: suspension/issued request が確定した後に driver が別呼出しで結果を返し、declared completion Operation が同じ WorkflowHandle の Workflow を再開して terminal outcome を返す。
- [ ] P1-10: 結果の重複、不正 identity/revision/snapshot/type/evidence、Failed/Cancelled outcome、timeout の未確認状態を focused specs で確認する。

## Stage 4 — Demonstration and acceptance evidence

Stage Status:
- Current status: OPEN
- Owner: TEAI
- Update rule: 下記 checklist と再現手順・結果を更新する。全項目完了が closure basis。

- [ ] P1-11: 一つの再現可能な起動手順で EVENT → Job/Workflow → INVOCATION → DELEGATION → CONTINUATION → 同一 Workflow の終端を示し、相関と業務結果を観測できる。
- [ ] P1-12: 下記受入例が実装と対応し、実施した検証・依存版・driver の性質・未検証保証を記録した。実装された CAR には CAR lint を適用する。

## Acceptance examples

| ID | Given | When | Then |
| --- | --- | --- | --- |
| AC-01 | 有効な FileArrived と固定 Binding | 公開 ingress を呼ぶ | CNCF Job ID と一つの WorkflowHandle が得られ、source/event から辿れる |
| AC-02 | 受領済み event | 同じ入力を逐次再送する | 新しい起動をせず同じ対応を返す。Binding 更新後も元の version を保持する |
| AC-03 | 不正 schema/source/Binding または同一キーの変更入力 | 受領する | 理由付き拒否。既存入力・Workflow を書き換えない |
| AC-04 | 起動した Workflow | 固定読取を実行し goal を委譲する | INVOCATION と DELEGATION が区別され、commit 前に claimable WorkOrder を外へ出さない |
| AC-05 | 確定した WORK_ORDER | driver が別呼出しで正当な結果を返す | 同じ instance identity を保持し、CNCF closing Action を一度実行して終端へ進む |
| AC-06 | 未完了または処理済み WorkOrder | stale/別 instance/不正型/不足 evidence、または重複結果を送る | 不正・重複処理で Workflow を進めず、claim token を外部へ出さない |
| AC-07 | 作業失敗または中止の typed result | completion を呼ぶ | 同じ Workflow が定義済み failure/cancel 結果を記録し、業務成功と表示しない |
| AC-08 | transport timeout または確定状況が不明な応答 | 同じ identity で確認する | 未確認と失敗を区別し、新規 goal/Workflow の自動作成をしない |
| AC-09 | 起動 Job が終了し Workflow が中断中 | 状況を照会する | Job 成功を業務完了と表示せず、現在の WorkOrder/Workflow を示す |

Executable specs では Given/When/Then を実際の setup/action/expectation の境界に置く。
文書文字列や Phase 完了印だけを検査する test は作らない。

## Completion and next action

全 checklist と受入例の証拠が揃った時点で Phase 1 の完了を判断する。
fake Workflow/Job だけの実証、単一呼出し内だけの callback、別 Workflow を起動する
「再開」は完了条件を満たさない。上流 capability が不足する場合は P1-04 など該当項目を
未完了として具体的な owner/不足契約を記録し、TEAI 内で代替 engine を実装しない。

次の作業は P1-03/P1-04。まず最小 fixture と利用する公開起動・completion 経路を
確定する。SBT は serialized runner、Cozy は対応 runner を使う。検証用 runtime
生成物は `target/` に置き、外部通信や実 OpenClaw 接続を成立したと推測しない。
