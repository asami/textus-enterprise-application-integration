# 最小 TEAI Integration Contract

Date: 2026-10-02
Contract identity: `teai.minimum-integration.v1`
Status: Phase 1 契約確定。実装・生成 ABI・store/provider・実行証明は未完了。
Owner: textus-enterprise-application-integration

本書は Phase 1 の規範的な振舞い契約である。[RULE.md](../../RULE.md) と
[文書 lifecycle](../../ai/directive/core/document-lifecycle.md) に従い、
[設計境界](../design/minimum-teai-integration-boundaries.md) と対応する。
進捗は [canonical checklist](../phase/phase-1-checklist.md)、検討履歴と固定した
CNCF の13参照は [旧仕様検討案](../notes/minimum-teai-integration-contract.md) に置く。
本書の logical Operation identity は TEAI が要求する契約であり、利用可能な生成 API
や production provider の存在を宣言するものではない。

## 1. 実証範囲と型語彙

構成済みの trusted な非 network harness、一つの runtime、逐次配送、一つの固定文書
fixture と Binding を対象とする。実 CNCF managed Job と typed Workflow を開始し、
委譲後の別呼出しで同一 WorkflowInstance を再開する。

`EnterpriseEvent` の schema は `teai.enterprise-event.v1` とする。

| Field | 型・意味 |
| --- | --- |
| `schemaVersion` | `String`。正確に `teai.enterprise-event.v1` |
| `sourceIdentity` | 構成済みの空でない source identity |
| `eventId` | source が発行する空でない stable identity |
| `eventType` | `FileArrived` |
| `occurredAt` | `java.time.Instant` |
| `payload` | `FileArrived(documentId, documentRevision)` |
| `provenance` | `VersionedReference(identity, revision)` |

配送は `DeliveryAttempt(attemptId, receivedAt)` として Event と分離する。
配送試行は event 等価性にも重複排除キーにも含めない。
キーは `(sourceIdentity, eventId)`。受理した typed Event の全 field を直接比較する。
内容ハッシュ、generic JSON 正規化、trace identity による代替は行わない。

| 固定 fixture 項目 | 値 |
| --- | --- |
| `sourceIdentity` | `fixture-document-source` |
| `documentId` | `fixture-document` |
| `documentRevision` | `document-v1` |
| endpoint | `fixture-document-read` |
| `bindingId` | `fixture-document-summary` |
| `bindingVersion` | `1` |

`VersionedReference(identity, revision)` は版付き参照の語彙である。
completion の文書 fact と evidence は `fixture-document` / `document-v1` の
正確な参照を必要とする。fixture 文書の内容と executable fixture の実体は後続実装で
対応付け、ここで取得済み・実行済みとは扱わない。

## 2. IntegrationBinding と宣言済み Operation

`IntegrationBinding` は次の field を持つ。

| Field | 契約上の値・役割 |
| --- | --- |
| `bindingId` | `fixture-document-summary` |
| `bindingVersion` | `1`。受領後も固定 |
| `sourceIdentity` | `fixture-document-source` |
| `eventType` | `FileArrived` |
| `inputTypeIdentity` | `teai.file-arrived.v1` |
| `componentIdentity` | `EnterpriseApplicationIntegration` |
| `startOperation` | `SummaryWorkflowService.startSummary` |
| `workflowIdentity` | `DocumentSummary` |
| `workflowRevision` | `document-summary-v1` |
| `inputMappingRevision` | 選択した入力 mapping の版 |
| `deterministicEndpoint` | `fixture-document-read` |
| `delegationEndpoint` | harness 構成で固定した委譲 endpoint |
| `resultTypeIdentity` | `teai.goal-outcome.v1` |
| `resultContractRevision` | 選択した result contract の版 |

schema/source/input と構成済み scope をまず検証する。新規 admission は一致する
Binding が正確に一つでなければ Job/Workflow 作成前に拒否する。ゼロ一致と複数一致を
理由付きで区別する。既存 receipt を新しい Binding 選択より先に参照し、同じ Event
の再送は Binding 更新後も元の version と association を維持する。

| Logical Operation identity | 責務 |
| --- | --- |
| `IntegrationService.receiveEvent` | Event admission と既存対応の再返却 |
| `IntegrationService.inspectEvent` | 送信前から分かる source/event key による読取 |
| `SummaryWorkflowService.startSummary` | managed Command に結び付ける明示的 typed Start |
| `SummaryWorkflowService.completeSummary` | 既存 CNCF result envelope による別呼出し completion |
| `DocumentEndpoint.retrieveFixtureDocument` | 宣言済み capability による固定文書読取 |

照会入力は事前に分かる構成済み `sourceIdentity` と `eventId` のみで成立する。
既知の continuation 参照は結果照会を絞れる。Job ID/Handle を必須にせず、読取は
決して dispatch しない。Cozy の実宣言/生成 ABI と CNCF managed-command、typed-start、
completion/store の binding は P1-04/P1-05 で証明してから実行成立を主張する。
手作成の generated metadata、fake Job/Workflow を証拠にしない。

## 3. EventReceipt、記録順序と観測

`EventReceipt` の field は `sourceIdentity`, `eventId`, `admittedEvent`,
`bindingId`, `bindingVersion`, `startInvocationReference`, `idempotencyKey`,
`jobIds`, `workflowHandle`, `knownContinuationReferences`, `confirmedGoalOutcome`,
`observation` とする。未確認の値を確定値として埋めない。
TEAI は既存 CNCF Entity/store policy を通す最小 metadata のみを所有する。
独自 Workflow DB、receipt engine、scheduler、recovery 実装を追加しない。

初回の新規 admission は次の順序を守る。

1. 入力と固定 trusted scope を検証する。
2. 受領キー、全 Event、固定 Binding、明示的 start invocation/idempotency 参照を記録する。
3. CNCF managed start Command を dispatch する。
4. 実 Job identity、typed WorkflowHandle、確認できた観測を同じ対応へ追記する。
5. 照会で association を取得できる状態にしてから開始成功応答を返す。

| Observation | 確認できた意味と再送の扱い |
| --- | --- |
| `Absent` | 照会可能な receipt がない。送信後なら未実行の証明にはならない |
| `PendingStart` | 受領記録はあるが起動結果待ち。再送で新規 dispatch しない |
| `KnownStarted` | 実 Job/Workflow association が確認できる。既知 identity と現在状況を示す |
| `Unconfirmed` | timeout や追記失敗等で開始/完了が確定できない。既知参照と不足する確認を示す |
| `Unavailable` | 照会不能。記録なしとは区別する |

初回新規 admission と送信後の回復照会は区別する。receipt 欠落は同じキー/別キーでの
自動再起動を許可しない。pending/unconfirmed receipt の再送も再 dispatch しない。
receipt と CNCF start が同一 transaction であると主張しない。既存 CNCF 照会で確認した
事実だけを加え、解決できなければ未確認を維持する。試験中は対応 key を保持する。
利用 dependency の実 `WorkflowInstancePersistence`/provider、issued WorkOrder 保存、
順序を守る suspension の証明が必要であり、interface や test probe だけでは足りない。

## 4. 四つの interaction

| Interaction | Direction | 実行契約 |
| --- | --- | --- |
| `EVENT` | Edge → TEAI | Event を検証し、一つの実 Job/typed Workflow を開始 |
| `INVOCATION` | TEAI → Edge | TEAI が固定文書 Operation を選び、ORCHESTRATION で宣言済み capability を呼ぶ |
| `DELEGATION` | TEAI → Edge | CNCF が suspension と issued `WORK_ORDER` を commit してから返す。Edge は固定 scope 内で達成手段を選ぶ |
| `CONTINUATION` | Edge → TEAI | 後の別呼出しで既存 CNCF result envelope を declared completion に返す。CNCF admission と fresh UoW/closing Action で元の Handle を再開 |

fixture は `WORK_ORDER → TERMINAL` のみを扱う。四 interaction は CNCF continuation
kind と別分類である。`DECISION`/`WAIT` を自動実行指示として解釈しない。
未対応 kind は明示して停止する。Presentation の文章から次操作を推測しない。
単一関数の同期 callback や別 Workflow の start は同一 instance 再開の証明にならない。

## 5. GoalOutcome と completion payload

`GoalOutcome` は閉じた application 型とする。

| Variant | 必須の意味 |
| --- | --- |
| `Succeeded` | summary、文書 fact、evidence references |
| `Failed` | `errorCode`, `message`、文書 fact、evidence references |
| `Cancelled` | `reason`、文書 fact、evidence references |

summary は `Succeeded` のみ必須。全 variant に正確な `document-v1` 文書参照を
completion facts と evidence として要求する。失敗/中止も truthful な fixture evidence を
供給し、成功へ変換しない。application payload codec は既存 `WorkflowProtocolV1`
`TypedValue`/`typeIdentity` と Start/Result codec の内側で閉じる。
generic CNCF envelope に独自 status field を追加しない。
CNCF closing Action が typed GoalOutcome を伴う terminal を選択する。
Workflow terminal/Job 成功は `Failed`/`Cancelled` の業務成功を意味しない。
cancelled fixture は返却 outcome であり、worker の強制停止 API を示さない。

## 6. Correlation、重複、failure

保持する chain は source/event → pinned Binding → admitted start invocation →
実 Job/WorkflowHandle → `runId`/`continuationId`/`expectedRevision`/`contextSnapshot` →
受理した facts/evidence/result → 同じ instance の terminal とする。
定義 revision と実行 revision、Event と delivery/invocation/continuation identity を分ける。
result は発行された値を引き継ぎ、identity を置換しない。CNCF が相関、revision、snapshot、
type、facts、evidence を検証し、一度だけ consume する。
private claim は serialize、返却、log、payload からの再構築のいずれも禁止する。

| ケース | 要求する振舞い |
| --- | --- |
| 等価 Event | 元の association を返す。開始済み/完了済みとも追加 start なし |
| 同一キーの変更 Event | conflict 拒否。元の入力/Binding/Workflow を上書きしない |
| 同じ決定的 invocation/WorkOrder | 既知の work/result を実証セッション内で再利用。新 goal/continuation を発行しない |
| 同じ result | 二つ目の Workflow/closing Action なし。duplicate 拒否だけで前回成功と判断しない |
| 異なる result・stale・wrong identity | 拒否し有効な提出余地、正当な suspension と既存結果を維持 |
| schema/source/Binding 不正・明確な start 拒否 | 構造化した理由を返す。受領と start 成功を分ける |
| timeout/応答喪失/追記失敗 | 既知事実と未確認を返す。業務 failure/cancellation の成立とはみなさない |
| type/facts/evidence 不足 | admission 拒否。業務完了扱いにせず正当な result を後で提出可能 |

cross-process exactly-once、同時 admission、外部副作用の retry 保証は持たない。

## 7. Trusted harness の authority

bootstrap 構成が source/endpoint/work scope を固定する。ingress、inspection、completion
は同じ制限された configured context を使う。payload source、Workflow selector、return URL
の自己申告は authority を成立させない。相関一致と claim 非公開は主体認可の代替ではない。
Phase 1 に network listener/live endpoint を設けない。構成と範囲外 misuse の拒否を証明する。
将来の外部公開には既存 CNCF identity/Operation 認可を結び、無権限の result/query の拒否後に
正当な completion が成立する追加証明を必要とする。harness の実績で network 認可を主張しない。

## 8. 受入と将来の Executable Specification 対応

下表は全て後続の Executable Specifications と実 dependency の観測で証明する。
対応先は P1 ledger の項目であり、本 Slice では Scala spec は作成しない。
将来の spec は `AnyWordSpec`、各 `in` の setup/action/expectation に隣接する
Given/When/Then、`should` matcher、ScalaCheck を用いる。文書文字列や完了印を検査する
test を受入証拠にしない。

| ID | Given / When | Then（観測する性質） | 後続証拠 |
| --- | --- | --- | --- |
| AC-01 | 有効 Event/Binding / receive と事前 key 照会 | 実 managed Job と一つの typed Workflow に解決 | P1-04/05/06 |
| AC-02 | 受領済み / 等価再送と Binding 更新後再送 | identity/version と起動数を維持 | P1-07 |
| AC-03 | schema/source 不正、Binding ゼロ/複数、同一キー変更入力 / admission | 理由付き拒否、start/上書きなし | P1-07 |
| AC-04 | 開始 Workflow / deterministic invocation、delegation | 固定読取を経て suspension/issued WorkOrder commit 後だけ発行 | P1-08 |
| AC-05 | 確定 WorkOrder / 別呼出し completion | 同一 instance を再開し closing Action は一度 | P1-09 |
| AC-06 | 未完了/処理済 work / stale、wrong instance/revision/snapshot/type/facts/evidence、重複 | 不正 advance なし、private claim 非公開、拒否後の正当結果は成立 | P1-10 |
| AC-07 | Failed/Cancelled / completion | truthful evidence を保持し業務 non-success | P1-10 |
| AC-08 | 応答喪失/未確認 / 下記三例 | 三例全てを closure basis にする | P1-10 |
| AC-08a | start と association 記録後の初回応答を捨て driver に Job/Handle なし / source/event 照会と再送 | 元の実 Job/Workflow、実起動数不変 | P1-10 |
| AC-08b | closing 後の応答を捨てる / source/event・既知 continuation 照会と retry | 確認済み結果、Workflow/closing 数不変。duplicate 拒否のみでは不足 | P1-10 |
| AC-08c | 送信後 Absent/PendingStart/Unconfirmed/Unavailable を個別に用意 / 照会 | 観測を区別し自動 restart なし | P1-10 |
| AC-09 | start Job done、Workflow suspended / inspection | Job 成功を業務完了にせず現在の Workflow/work を示す | P1-06/10/11 |
| AC-10 | 固定 configured scope / scope misuse と別呼出し inspection/completion | payload 自己申告を拒否、構成上非 network、許可 scope のみ | P1-04/10 |

P1-11 は四 interaction の再現可能な一本の手順、P1-12 は全 AC と実行結果・依存版・
未検証保証の対応を要求する。P1-04..12 は未完了である。

## 9. Non-goals

実 OpenClaw/LLM/FTP/SFTP/Workspace、multiple workers/同時受領/crash recovery/
distributed exactly-once、独自 Workflow/claim/receipt/recovery engine、他 repository source
編集、新認証 service、push/publication/deployment、後続 Phase と split child は対象外。
dependency capability が不足すれば owning repository と不足 edge を示して停止し、
TEAI の代替 engine を実装しない。
