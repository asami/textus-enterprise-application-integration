# Minimum TEAI Integration Contract — CNCF alignment proposal

Date: 2026-10-02
Status: specification proposal; implementation and executable acceptance pending
Owner: textus-enterprise-application-integration
Plan: [Phase 1](../phase/phase-1.md)
Direction: [Architecture journal](../journal/2026-09-21-teai-architecture-and-goals.md)

## 1. 目的と最初の実証範囲

TEAI の最小境界を Enterprise Event、Integration Binding、外部との四つの
interaction として定め、一つのイベントから CNCF Job を経て Workflow を開始し、
外部への goal 委譲と結果受理によって同じ WorkflowInstance を再開する。

これは仕様検討案である。以下の TEAI 名称は既存 API の宣言ではない。Phase 1 の
契約確定で design/spec と executable specification に対応付ける。
CNCF の protocol、Job、Workflow、claim、UnitOfWork を TEAI で再定義しない。

最初の実証は単一 TEAI runtime、逐次配送、固定された一つの Binding と
一つの Workflow definition を対象とする。外部側は OpenClaw の役割を担う
決定的な契約テスト用ドライバを使用する。CNCF の Job、WorkflowInstance、
Continuation、結果検証と再開は実実装を通す。実 OpenClaw の API/認証、
実 FTP/SFTP、Google Workspace、LLM の利用をこの実証の成立条件にしない。
テスト用ドライバの成功を実 OpenClaw 接続の成功とは扱わない。

## 2. 確認した CNCF 契約

調査対象: `goldenport-cncf` HEAD
`f7cf5bc04c11b9f74e09d61b8199452c5edb3275` と現在のローカルファイル。
Job 関連には並行作業中の変更がある。これはソース調査結果であり、依存版の
採用・公開済み artifact・今回のテスト成功を意味しない。実装開始時に利用版を固定する。
参照パスの `dev????` は環境依存であり、repository identity を先に解決する。

| 項目 | 確認できた契約と TEAI への含意 |
| --- | --- |
| Managed Job | `JobEngine.submit`、status/result/query、管理された Command 実行がある。plain `Sync` は Job を作らないため、実証の起動経路で Job ID を観測する |
| Typed start | `WorkflowProtocolV1.WorkflowStartRequest` は `startOperation`、definition identity/revision、`TypedValue`、`invocationReference`、`idempotencyKey` を持つ。profile が選んだ明示的な Start Operation 用であり、任意の Workflow を外部から選べる汎用 start API ではない |
| 同一 Workflow | `WorkflowHandle` は component、definition identity/revision、instance identity、protocol version。実行時の `expectedRevision` は別項目であり definition revision と混同しない |
| Continuation | `WORK_ORDER / DECISION / WAIT / TERMINAL` の閉じた型。TEAI の四つの interaction kind とは別の分類 |
| Goal 委譲 | `WORK_ORDER` の `ContinuationRequest`、typed input/result、ContextBundle、Completion/Evidence contract と ExecutionRequirement を利用する |
| 外部結果 | `ContinuationResult` は handle、runId、continuationId、expectedRevision、contextSnapshot、typed result、resultReference、completionFacts、executionEvidence を持つ |
| 結果検証 | `admitResultC` は発行済み WorkOrder との identity/revision/snapshot/type/facts/evidence 対応を検査する。業務 payload の意味検証は application codec/adapter が所有する |
| Adapter | `ContinuationSpiAdapter` は外部入力を typed result へ変換する。claim token、StateMachine route、UoW commit を所有しない |
| 公開 completion | `WorkflowCompletionServiceOperation.bindC` は declared Provided Operation、保存済み WorkOrder/instance、closing program を結び付ける。公開されている completion 経路を利用する |
| 保存と再開 | `ContinuationRuntime` は commit 後の suspension 公開、private claim、fresh UoW での resume を提供する。completed continuation の再 resume は拒否される。重複応答の再返却はこの事実だけでは保証されない |
| 原子性 | `WorkflowInstanceAtomicTransitionV1` は opt-in capability の要求。interface の存在は provider の原子性の証明ではない。現在の completion 経路には resume 後の instance append があるため、全境界の一括 transaction を仮定しない |

### Job と Workflow の対応を混同しない

既存 `WorkflowEngine.handle` は event/entity/status に基づいて Action を Job に
submit し、独自の WorkflowInstance と relatedJobIds を記録する経路である。
`JclRuntimeBridge.submitWorkflow` はその registration/entrypoint を利用する。
これを typed durable `WorkflowProtocolV1` の汎用 start/resume と同一視しない。

推奨する検証順は、TEAI の明示的な起動 Operation を managed Command/Job で
実行し、そこで生成された typed Workflow の Start に結び付けることである。
結果受理は別の公開 completion Operation を通し、同じ WorkflowHandle に戻す。
実際の bootstrap、生成 ABI、管理 Command の組合せは Phase 1 の最初の接続検証で確認する。

Event と一つの WorkflowHandle の対応に、起動 Job ID と、存在する場合は
後続処理の Job ID を関連付ける。一つの Job が外部待機期間全体を保持するという
保証は追加しない。起動 Job の成功は WorkOrder を返せたことを表し得るので、
業務の完了判定には Workflow の terminal と typed outcome を使う。
将来、一つの enterprise Job に全期間を集約する場合は CNCF 所有の契約として検討する。

### ソースと仕様の参照

- [JobEngine](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/job/JobEngine.scala)
- [Managed Command use cases](../../../goldenport-cncf/docs/overview/use-cases/jobs/README.md)
- [WorkflowEngine](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowEngine.scala)
- [JCL runtime bridge](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/component/builtin/jobcontrol/JclRuntimeBridge.scala)
- [WorkflowProtocolV1](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowProtocolV1.scala)
- [Start codec](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowStartJsonV1.scala)
- [Result codec](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowResultJsonV1.scala)
- [Continuation runtime](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/ContinuationRuntime.scala)
- [Continuation SPI adapter](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/ContinuationSpiAdapter.scala)
- [Completion Operation](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowCompletionServiceOperation.scala)
- [Atomic transition capability](../../../goldenport-cncf/src/main/scala/org/goldenport/cncf/workflow/WorkflowInstanceAtomicTransitionV1.scala)
- [Workflow protocol executable specification](../../../goldenport-cncf/src/test/scala/org/goldenport/cncf/workflow/WorkflowProtocolV1Spec.scala)
- [Continuation runtime executable specification](../../../goldenport-cncf/src/test/scala/org/goldenport/cncf/workflow/ContinuationRuntimeSpec.scala)

古い generic JSON の概念例を新しい wire schema として複製しない。
現行 codec は `schemaVersion = cncf.workflow-protocol.v1` と
`typeIdentity/value` を使う。CNCF 型は JVM 内では直接合成し、外部境界では
既存 codec と TEAI/application payload codec を組み合わせる。

## 3. Enterprise Event と Integration Binding

以下は TEAI 所有の最小意味項目。正式な型名・field 名は contract freeze で確定する。

| 対象 | 必要な情報 |
| --- | --- |
| Enterprise Event | event schema/version、認証済み source identity、source が発行した stable eventId、event type、occurredAt、typed payload または版付き参照、provenance |
| 配送試行 | delivery/attempt identity、receivedAt、必要な transport 診断。再送ごとに変わってよいが event identity は変えない |
| Integration Binding | binding identity/version、許可する source/event type/input type、固定した CNCF component/Start Operation/definition revision、入力 mapping、endpoint capability/policy、対応する result contract |
| 受領対応 | source/event key、受理した event、選択済み binding version、start invocation reference/idempotency key、Job IDs、WorkflowHandle、受領・結果確認の観測 |

初期キーは `(sourceIdentity, eventId)` とし、業務 identity を内容ハッシュから作らない。
認証主体と sourceIdentity の対応は ingress binding が検証する。payload にある
自己申告の source、Workflow selector、return URL を実行権限として採用しない。
初期 fixture は構成済みの一つの source と endpoint のみを使用する。

Binding 解決は明示的な選択を一つ返す。未一致・複数一致・未対応 schema は
Job/Workflow を開始する前に理由付きで拒否する。既存キーの再送は、先に
受領対応を参照し、Binding 更新後も元の version と Workflow を返す。
管理情報は標準 CNCF Entity/persistence で必要最小限に表現し、TEAI 独自の
Workflow 状態 DB、配送エンジンや lock/recovery framework を作らない。

## 4. 四つの interaction と CNCF への対応

| Interaction | Direction | 最小契約と実行意味 |
| --- | --- | --- |
| EVENT | Edge → TEAI | Enterprise Event を受領・検証し、Binding に従って一つの Workflow を起動する。新しい source event と既存委譲の返却を混同しない |
| INVOCATION | TEAI → Edge | invocation identity、固定 capability、typed input/result、許可範囲を渡す。具体的な操作は Textus が選択済み。Required Operation の ORCHESTRATION として結果を受ける |
| DELEGATION | TEAI → Edge | 発行済み CNCF WORK_ORDER と application goal/input/result contract を渡す。Edge は許可範囲内で達成手段を選ぶ。実行前に suspension と issued WorkOrder が確定していること |
| CONTINUATION | Edge → TEAI | 発行済み WorkOrder に対応する CNCF ContinuationResult を返す。adapter が payload を検証し、CNCF が相関・revision・snapshot・evidence を検証して既存 Workflow を再開する |

TEAI が logical authority、Edge が委譲作業の physical execution loop を持つ。
DELEGATION は必ずしも TEAI からの unsolicited push を意味しない。初期 driver は
TEAI 呼出しの応答として WorkOrder を受け取り、実行し、completion Operation を
呼び戻す。以後の continuation も構造化された kind に従って扱う。

四つの interaction kind を CNCF の continuation kind に一対一変換しない。
`DECISION` は人の判断、`WAIT` は待機であり、goal を自動実行する指示ではない。
最初の実証は `WORK_ORDER → TERMINAL` を通す。driver は未対応 kind を明示して停止し、
Presentation の文章を parse して次の処理を推測しない。

OpenClaw の API、endpoint、認証方式、モデル名を core に固定しない。
実 OpenClaw 用 binding は、利用版と実 API を確認した後に endpoint adapter として追加する。
実行手段は Program → Local LLM → Frontier AI の順で選び、最初の契約検証には
生成モデルを必要としない。モデル・worker・mapping policy は実行 evidence であり
Workflow の状態遷移条件ではない。機密 payload は許可された最小参照に絞り、
credential や private claim token を WorkOrder、結果、audit に出さない。

## 5. Correlation と重複配送

保持する chain は次の通り。trace/correlation ID 自体を重複排除キーにしない。

```text
source + eventId
  -> bindingId/version + admitted start invocation
  -> start Job ID -> WorkflowHandle.instanceIdentity
  -> runId + continuationId + expectedRevision + contextSnapshot
  -> returned result reference + execution evidence
  -> same WorkflowHandle + terminal outcome
```

結果側の handle/run/continuation/revision/snapshot は発行値を引き継ぐ。
source event、delivery attempt、deterministic invocation、Workflow instance、
continuation は別の identity として扱う。definition revision と実行 revision も分ける。

| ケース | 初期の扱い |
| --- | --- |
| 同じ event の逐次再送 | 保存済み対応を返し、二つ目の Job/Workflow を開始しない。開始・完了済みのいずれでも新規 event にしない |
| 同一キーで異なる入力 | 最小 fixture の typed fields / versioned references を直接比較して conflict を返す。元の入力・Binding・Workflow を変更しない。汎用 JSON 正規化やハッシュ機構は不要 |
| 開始したか不明 | 対応と CNCF の観測結果を照会し、未確認として返す。別キーで自動再実行しない。`WorkflowStartRequest.idempotencyKey` の存在だけで原子的 start を保証しない |
| 同じ INVOCATION の再配送 | 決定的な読取 fixture で確認する。外部副作用の再試行は provider の保証が確認されるまで自動化しない |
| 同じ WORK_ORDER の再配送 | 同じ委譲として扱い、driver は既知の結果を再利用する。新しい goal や continuation を発行しない。初期 driver の保証はその実証セッション内に限定 |
| 同じ CONTINUATION 結果の再送 | CNCF の consumed/duplicate 判定を通し、closing Action を再実行しない。成功結果の replay が確認できない場合は duplicate/conflict と現在の照会可能情報を返す。単なる失敗を「前回成功」の証明にしない |
| 異なる結果・stale revision・別 Workflow | 受理せず、現在の suspension と元の結果を維持する。外部の申告だけで claim を再作成・変更しない |

初期実証は逐次配送で確認する。multi-worker、同時受領、crash をまたぐ
exactly-once 外部副作用、無期限 deduplication、再送 scheduler は保証しない。
試験期間中は受領対応を保持し、途中で key を破棄しない。保持期限・削除後の再送・
並行 admission は、本番 transport/storage の選定と合わせて別途仕様化する。

## 6. Failure model

| Failure | 観測・処置 |
| --- | --- |
| schema / source / Binding 不正 | 入力拒否。起動 Job/Workflow を作成したと報告しない |
| 明確な起動拒否 | 構造化した理由を返す。受付と Workflow 開始成功を分ける |
| transport timeout / 応答喪失 | 結果未確認。failure や cancellation の成立とはみなさず、同じ identity で照会する |
| goal の failed / cancelled | application-owned `GoalOutcome` の typed result として受理可能な範囲を定義し、CNCF が選ぶ failure/cancel 終端へ進む。証拠を捏造して successful outcome に変換しない |
| result の type / facts / evidence 不足 | admission 拒否。正当な結果の提出余地を維持し、業務を完了扱いにしない |
| commit 後の結果確認・history append 失敗 | 確認できた事実と未確認部分を返す。新規 Workflow を起動せず、CNCF の recovery/照会契約へ戻す |

`GoalOutcome` は例えば `Succeeded / Failed / Cancelled` の閉じた application 型とする。
失敗時に要求できる facts/evidence も completion contract に定義する。
この outcome は generic CNCF envelope に独自の status field を足す理由にはならない。
失敗結果を正常に処理して Workflow が terminal になった場合も、業務成功とは表示しない。
初期の cancelled fixture は Edge の作業結果であり、実 worker を強制停止する API の証明ではない。

## 7. 一つの実証 Workflow

題材は小さな固定文書の `FileArrived` fixture と、その文書の要点・根拠の返却とする。
KnowledgeHub Admission や TKL の canonical Candidate を新設しない。

```text
FileArrived fixture (EVENT)
  -> TEAI validates source/schema and selects pinned Binding
  -> CNCF managed start Job -> explicit typed Start -> WorkflowInstance W
  -> RetrieveFixtureDocument (INVOCATION / ORCHESTRATION, read-only)
  -> CollectDocumentSummary goal (DELEGATION / WORK_ORDER)
  -> persist suspension + issued request; yield to deterministic Edge driver
  -> driver returns GoalOutcome + result/evidence references (CONTINUATION)
  -> declared completion Operation -> admission -> fresh UoW / closing Action
  -> same WorkflowInstance W -> terminal typed outcome
```

WorkOrder の発行と結果受理は別呼出しにする。単一関数内の同期 callback だけで
中断・再開の証明としない。driver の goal result は fixture でよく、実証の焦点は
AI の回答品質ではなく contract、authority、identity、resume である。

## 8. 実装前に確定する項目

1. Cozy の現行 CML/生成 ABI と CNCF dependency coordinate、TEAI component の最小 bootstrap。
2. managed Command → typed Start の公開経路と観測できる Job/Workflow の対応。
3. TEAI receipt metadata に使用する既存 Entity/store policy と初期の逐次実行範囲。
4. goal input/result codec、success/failure/cancel の closing program と required evidence。
5. 実証の公開起動・completion・照会 Operation と固定 driver の呼出し方法。

未提供の framework capability は owner を CNCF/Cozy として具体的な不足を返す。
TEAI の独自 workflow engine、private claim 操作、汎用 HTTP client、独自 persistence/
recovery に置き換えない。これらの決定と focused proof を得てから当該実装を進める。
