# 組織横断の決定（ADR）

jazmf-dx の **アプリをまたぐ** 判断を置く。1 つのアプリに閉じる判断は、これまでどおり各アプリの `decisions/` に置く。

> 置き場所について: Web アプリをモノレポに寄せる決定（[ADR-0001](adr-0001-web-apps-monorepo.md)）により、
> ここにある ADR はモノレポができた時点で `decisions/` ごと移す。それまでは ai-dev-platform が
> 「アプリから見た唯一の入口」なので、ここに置く。

形式は Governance §1.1 のとおり（ステータス／コンテキスト／決定／結果）。番号は連番で、欠番を埋めない。
置き換えたときは古いほうのステータスを更新して残す。

| 番号 | 主題 | 日付 | ステータス |
|---|---|---|---|
| [0001](adr-0001-web-apps-monorepo.md) | Web アプリをモノレポに寄せる | 2026-11-05 | 採用 |
| [0002](adr-0002-pr-gates-runner-and-staging.md) | PR ごとの検査・self-hosted runner・ステージングの自動化 | 2026-11-05 | 採用 |
| [0003](adr-0003-ui-screen-types-and-shared-shell.md) | UI は画面の型とナビを正にし、共通シェルを配る | 2026-11-05 | 採用 |
| [0004](adr-0004-agent-working-agreement.md) | エージェントへの指示の持ち方（約束・CLAUDE.md の上限・検査化・セッションの単位） | 2026-11-05 | 採用 |

背景と進め方の全体は [docs/direction-2026-11.md](../docs/direction-2026-11.md)。
根拠となった調査は [dx-portal/docs/reviews/2026-11-05-platform-review.md](https://github.com/jazmf-dx/dx-portal/blob/main/docs/reviews/2026-11-05-platform-review.md)。
