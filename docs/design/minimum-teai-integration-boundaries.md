# 最小 TEAI Integration の設計境界

Date: 2026-10-02
Status: Phase 1 の責務・順序決定。dependency binding と実行証明は未完了。
Contract: [teai.minimum-integration.v1](../spec/minimum-teai-integration-contract.md)
Progress: [canonical checklist](../phase/phase-1-checklist.md)

[RULE.md](../../RULE.md) と [文書 lifecycle](../../ai/directive/core/document-lifecycle.md)
に従う安定した設計決定である。検討過程は [notes](../notes/minimum-teai-integration-contract.md)
に残し、本書を探索 diary にしない。

## 責務と所有者

| 所有者 | 所有する責務 |
| --- | --- |
| TEAI | EnterpriseEvent、IntegrationBinding、typed mapping/payload codec、interaction adapter、最小 receipt metadata、source/event 照会、相関と integration audit |
| CNCF | managed Command/Job、typed WorkflowInstance/Handle、Continuation、admission/private claim/revision、suspension、issued WorkOrder persistence、fresh UoW と closing Action、実行状態と再開 |
| Cozy | component/CML 宣言からの生成と実 ABI |
| Edge harness | 構成済み scope 内の物理実行 loop、固定文書 fixture、既知 work/result の実証セッション内再利用 |

TEAI は logical authority と Binding を持ち、Edge は委譲 scope 内の達成手段を選ぶ。
独自 Workflow DB、receipt 状態機械、claim 操作、scheduler、recovery framework に
置き換えない。receipt は既存 CNCF Entity/store policy 上の最小 metadata に限定する。
一つの Job が外部待機期間全体を保持する保証を追加しない。開始 Job と Workflow の
業務結果を分け、terminal の typed GoalOutcome で業務結果を示す。

## Interaction と順序

`EVENT` は admission、`INVOCATION` は TEAI が選択済みの deterministic Operation を
ORCHESTRATION で実行、`DELEGATION` は scope 内の goal 達成を委譲、`CONTINUATION` は
返却結果で同一 instance を再開する。これらを CNCF continuation kind と一対一対応させない。
最小実証は `WORK_ORDER → TERMINAL` とし、`DECISION`/`WAIT` 等の未対応 kind は停止する。

入力/scope 検証 → 全 Event・pinned Binding・start invocation/idempotency 参照を記録 →
managed CNCF start dispatch → 実 Job/Handle と観測を追記 → source/event 照会可能になって
から start 成功応答、という順序を固定する。既存 receipt を新 Binding 選択より先に見る。
receipt と start を一括 transaction とみなさない。pending/unconfirmed の再送と照会は
dispatch せず、送信後の Absent/Unavailable を未実行の証拠にしない。

固定文書の declared capability を呼び、CNCF が suspension/issued WorkOrder を commit
してから Edge に返す。Edge は後の別呼出しで既存 CNCF envelope を返す。
declared completion → CNCF admission → fresh UoW/closing Action → 元の WorkflowHandle
の terminal とする。同期 callback と新 Workflow start はこの境界を満たさない。
private claim は CNCF 内部に留め、payload、応答、log から生成・復元しない。
application codec は TypedValue/typeIdentity 内の GoalOutcome を扱い、generic envelope
に status を追加しない。Failed/Cancelled の正常な受理と業務成功は分ける。

## 依存の証拠を分ける

既存の [13固定参照](../notes/minimum-teai-integration-contract.md#ソースと仕様の参照)
は CNCF source 読取の証拠である。source commit、使用 artifact coordinate、実 Cozy 宣言/
generated ABI、focused runtime proof を別々に記録する。
未コミット Job 観測や cache artifact の存在を採用 dependency の能力証明にしない。
既存 WorkflowEngine/JCL の経路と typed durable WorkflowProtocolV1 の経路を混同しない。

P1-04/P1-05 は `IntegrationService.receiveEvent`/`inspectEvent`、
`SummaryWorkflowService.startSummary`/`completeSummary`、
`DocumentEndpoint.retrieveFixtureDocument` を実 component/生成 ABI と provider に結び付ける。
実 managed Job、typed Start、WorkflowInstancePersistence/provider、issued WorkOrder 保存と
順序付き suspension、completion と receipt store の利用可能性を証明する。
bare SPI/interface、test probe、手作成 generated metadata、fake Workflow/Job は証明にしない。
不足 capability は具体的な owner/依存 edge として返し、TEAI の local substitute を作らない。
CNCF/Cozy/sbt-cozy はこの Slice では読取専用である。

## Authority と保証範囲

bootstrap が source/endpoint/work scope を固定し、ingress/inspection/completion は
同じ restricted configured context を使う。payload source/selector/return URL は authority
ではない。Phase 1 は非 network harness の構成と misuse 拒否を証明する。
将来外部公開するときは既存 CNCF identity/Operation 認可、無権限 result/query 拒否後の
正当な completion を要求し、新 authentication service は設けない。

固定 fixture、一 runtime、逐次配送、同一 demonstration session だけを保証範囲とする。
response loss の start/closing 数不変を証明しても、crash/restart、distributed transaction、
cross-process exactly-once を保証しない。実 OpenClaw/LLM/FTP/SFTP/Workspace、multi-worker、
他 repository source 編集、push/publication/deployment、successor/split child は範囲外。
