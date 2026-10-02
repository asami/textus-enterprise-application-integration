# Phase 1 canonical checklist reference

本書は Phase 1 の canonical checklist 参照文書である。実際の完了状態は
[phase-1.md の inline Stage ledger](phase-1.md#stage-1--contract-and-dependency-binding)
へ委譲する。checkbox truth は同ファイルの P1-01..P1-12 のみとし、本書に mutable な
完了 ledger を複製しない。[共有 checklist 規則](../../ai/directive/core/phase-subphase-checklist.md)
が進捗意味を定める。

| Ledger の参照 | 正確な item IDs | Closure に必要な証拠 |
| --- | --- | --- |
| [Stage 1](phase-1.md#stage-1--contract-and-dependency-binding) | P1-01, P1-02, P1-03, P1-04 | directive、source 調査、規範契約/fixture、実 dependency/ABI/provider focused proof |
| [Stage 2](phase-1.md#stage-2--component-foundation-and-event-admission) | P1-05, P1-06, P1-07 | component 生成/build/spec、実 admission/lookup、重複と拒否 |
| [Stage 3](phase-1.md#stage-3--delegation-and-same-instance-resume) | P1-08, P1-09, P1-10 | deterministic invocation、commit 後委譲、別呼出し同一 instance 再開、異常系 spec |
| [Stage 4](phase-1.md#stage-4--demonstration-and-acceptance-evidence) | P1-11, P1-12 | 再現手順、全受入と依存版/driver/未検証保証の対応、実装 CAR lint |

[受入例 AC-01..AC-10](phase-1.md#acceptance-examples) と
[規範 spec の将来 executable-spec 対応](../spec/minimum-teai-integration-contract.md#8-受入と将来の-executable-specification-対応)
を併用する。AC-08 の closure basis は AC-08a、AC-08b、AC-08c の三例全てである。
P1-03 の文書証拠は [contract](../spec/minimum-teai-integration-contract.md) と
[design](../design/minimum-teai-integration-boundaries.md)。この文書確定は P1-04..12 の
実装・実行・依存能力を証明せず、Step/Stage/Phase の closure を意味しない。
