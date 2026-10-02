# Phase 1 minimum contract freeze

Date: 2026-10-02
Identity: PHASE-1 / P1-S1 / P1-S1-D1
Source invocation: `$cncf-goal-phase 1`（occurrence 1）
Status: documentary contract freeze; implementation and acceptance proof pending

利用者が採用した [既存 review clarification の反映](2026-10-02-phase-1-review-follow-up.md)
と [元の review](2026-10-02-phase-1-review.md) を保持したまま、parent が選択した frozen
Implementation Manifest に従い、[規範契約](../spec/minimum-teai-integration-contract.md) と
[設計境界](../design/minimum-teai-integration-boundaries.md) を確定した。
authority は `teai-phase1-authority-20261002` revision 1、goal binding は
`teai-phase1-goal-binding-20261002` revision 1、PLAN epoch は
`teai-phase1-initial-plan-epoch` revision 1。

parent の確定対象は `teai.minimum-integration.v1` の Event/Binding/Receipt/GoalOutcome、
五つの logical Operation identity、固定 document-v1 fixture、四 interaction、記録/dispatch/
lookup/応答順序、typed equality と重複/conflict、response loss の観測、trusted 非 network
harness の authority 境界である。13件の固定 source 引用は notes に保持する。
この記録自体は新しい振舞い authority ではない。

[canonical checklist](../phase/phase-1-checklist.md) は既存 inline ledger を唯一の checkbox
truth として参照する。P1-03 は契約/設計/fixture 語彙の文書証拠に対応する。
Phase は open、Stage 1 は IN_PROGRESS、P1-04..12 は未完了である。

| 未完了 item | 残る具体的証明 |
| --- | --- |
| P1-04 | source commit/artifact/生成 ABI の区別、managed start/completion/lookup の binding、trusted scope、実 persistence/provider と suspension/issued WorkOrder の focused proof |
| P1-05 | 現行 Cozy/CML component の生成/build と focused Executable Specifications |
| P1-06 | 実 managed Job/typed Workflow の起動と source/event association 照会 |
| P1-07 | admission 拒否、逐次再送、Binding 更新後の版固定、conflict 非上書き |
| P1-08 | deterministic 文書 INVOCATION と commit 後 WORK_ORDER DELEGATION |
| P1-09 | 別呼出し completion、fresh UoW/closing Action、同一 instance terminal |
| P1-10 | 不正相関/型/facts/evidence/重複、Failed/Cancelled、AC-08a/b/c、harness scope の executable proof |
| P1-11 | 四 interaction の一本の再現手順と業務結果観測 |
| P1-12 | 全 AC と実装/実施結果/依存版/driver/未検証保証の対応、実装 CAR lint |

本 Slice は文書編集のみである。code、SBT、ABI 生成、runtime 実行、commit、publication
の成立を記録しない。静的検証と独立 review、Step/Phase の遷移は parent が所有する。
