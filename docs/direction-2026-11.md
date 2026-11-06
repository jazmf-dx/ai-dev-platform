# 方針（2026-11）— 決めたことと進め方

2026-11-05 の全体レビュー（[dx-portal/docs/reviews/2026-11-05-platform-review.md](https://github.com/jazmf-dx/dx-portal/blob/main/docs/reviews/2026-11-05-platform-review.md)）を受けて決めた方針。
決定の本文は [decisions/](../decisions/) の ADR 0001〜0004。ここは全体像と順番だけを持つ。

## 前提（変わったこと）

| 以前 | いま |
|---|---|
| 手作業と justfile で運用、CI は無い | CI はあるが production 宛てだけ。justfile が運用の本体（デプロイ・cron・バックアップ） |
| Bedrock の従量課金。トークン節約のために共有資産とルールを作った | サブスク。規約の価値は「エージェントの出力を正しくする」「機械で止める」だけ |
| デザインシステムで UI の品質を保つ | 色と部品を縛っても画面は揃わず、縛られる場面だけが残った |
| Django は管理サイトが使えるから。自分が読めるから | 管理サイトはほぼ使わない。エージェントの速度で人はコードを読めない |
| ターミナルから claude | アプリ版。クラウドとローカルが混在し、指示が届かない |

## 原則

1. **アプリを 1 つ足すコストが一定で、既存を劣化させない。** 共通のものは写さず配る
2. **人はコードを読まない前提で設計する。** 人が見るのは決定（ADR）・画面（ステージング）・検査の結果
3. **文書で縛らず、機械で止める。** ルールは検査にできるものだけ残す
4. **エージェントの経路に必ずゲートがある。** クラウドでもローカルでも main に入る前に同じ検査が走る
5. **利用者には 1 つのシステムに見える。** 共通のシェル・戻り導線・SSO・用語

## 決めたこと

| 領域 | 決定 | ADR |
|---|---|---|
| リポジトリ | Web アプリをモノレポに寄せる。デプロイはアプリごと。段階的に移し、dx-portal は最後 | [0001](../decisions/adr-0001-web-apps-monorepo.md) |
| CI・ブランチ | PR 必須。PR は変えたアプリだけ、main で全部 + イメージ + ステージング自動デプロイ + スモーク。runner は self-hosted。production ブランチ廃止 | [0002](../decisions/adr-0002-pr-gates-runner-and-staging.md) |
| ステージング | 常に main が動いている場所。全アプリを 1 ホストに。本番 DB の週次コピーも cron。人が見るのは変えた画面だけ | [0002](../decisions/adr-0002-pr-gates-runner-and-staging.md) |
| UI/UX | 画面の型とナビを正にし、共通シェルを配る。部品は Kit、色は tone。Focus App は例外。IA 文書 1 枚 | [0003](../decisions/adr-0003-ui-screen-types-and-shared-shell.md) |
| エージェント | 約束 10 行を全 CLAUDE.md の先頭に。CLAUDE.md は 100 行以内。ルールは検査に。1 リポジトリ = 1 セッション。クラウド環境を定義。モードの使い分け | [0004](../decisions/adr-0004-agent-working-agreement.md) |
| 技術スタック | Django を続ける。変えるとしても衛星 1 本で試す。Go などの別言語は要らない（棚卸し: 重い処理はブラウザ内、サーバーの重さは待ち時間） | （レビュー §4.3・ADR-0005 コンテキスト） |
| バックグラウンド処理 | Procrastinate を全アプリへ横展開し、worker コンテナを足す。定期処理もそこで持ち、ホストの crontab を無くす。最終成功時刻を `/api/status/` に出す。Redis は入れない | [0005](../decisions/adr-0005-jobs-on-procrastinate.md) |
| 共通コードとパッケージ | Python は uv workspace、UI Kit は ui-platform を subtree で取り込み bun workspace。GitHub Packages への publish は外で使う人がいるときだけ。雛形 `just new-app` | [0006](../decisions/adr-0006-shared-code-and-ui-kit-in-monorepo.md) |
| ルールの置き場 | モノレポの `docs/` に畳む（原則 1 枚 + 圧縮した Standard + Playbook + Recommendations）。個人側 upstream への追従をやめ、3 リポジトリはアーカイブ。desktop-platform は desktop-apps へ | [0007](../decisions/adr-0007-rules-live-in-monorepo.md) |
| フロントの境界 | **保留**。別でしっかり議論する。それまでは Standard（テンプレートが土台・対話 UI は Islands・htmx は部分更新）を既定とする | — |
| 運用 | status を 1 か所が巡回。バックアップ・status・build info は雛形に入れる（定期ジョブの台帳は ADR-0005 で解決） | — |

## 進め方

| 段 | 期間の目安 | 内容 | 依存 |
|---|---|---|---|
| 0 | 1〜2 週間 | **誤誘導を止める**（dx-portal）: 約束ブロックを全 CLAUDE.md の先頭に、監査レポートを docs へ、Alpine の rules を既存限定に、agents 12 本削除、skills 3 本修正、ADR 番号重複の解消、CLAUDE.md の数値を参照に、クラウド環境の定義 | なし |
| 1 | 2〜3 週間 | **ゲートを置く**: self-hosted runner、PR の workflow（affected）、main の workflow（全部 + ステージング + スモーク）、`just check` を CI と同じ定義に、固定色・Alpine・`<select>` の「新規ゼロ」テスト、production ブランチの廃止 | runner のホスト |
| 2 | 1 か月 | **決めた撤去を実行**: surveys（22.9k 行）・market_prices と csvflow（10.2k 行・tools 本番切り替えと同時）・preview 10 本・日程調整の島・旧サイドバー 4 件・samples 17 件・完了した handoff と plans | 段 1（検査があってから消す） |
| 3 | 半日で決め、2〜3 週間で反映 | **IA を 1 枚で決める**: 分類表を 1 つに、`/manage/` の規則と巡回・タスクの移設計画、用語、global nav の基準、v1/v2 と surveys の撤去期限 | なし（決めるだけ） |
| 4 | 1〜2 か月（新規・改修から） | **画面の型を入れる**: `classes.css` の導入、一覧・詳細・ページヘッダーの型、h1 の二重解消、部品の正本を Kit に、`*_display_class` の tone 化、Focus App の線引き | 段 3 |
| 5 | 四半期 | **モノレポと共通パッケージ**: モノレポを作る → 共通パッケージ（logging・build_info・status・clientip・csvsafe・Directory client・共通シェル）→ 衛星を小さい順に subtree で移す → 雛形 `just new-app` → ai-dev-platform を畳み込む | 段 1（CI の形が決まってから） |
| 6 | 段 5 と並行 | **横断**: 共通シェルと戻り導線、SSO の連続性、standards-adoption の更新、Django の pin（tools・knowledge）、バックアップの無い衛星、status の巡回 | 段 5 の共通パッケージ |

段 0〜2 は新機能と並行できる。段 3 は決める作業で、飛ばすと段 4 が画面ごとの判断に戻る。段 5 は段 1 の形を前提にする。

## まだ決めていないこと

- **フロントの境界**（2026-11-06 に保留。別の回で議論する。論点: 画面単位の React をどこまで許すか、Alpine の撤去の速度、htmx 2 への統一、チャートの正本）
- **認証の一本化**（dx-portal のマジックリンク・Garoon 入口と衛星の authentik OIDC が並び、SSO がつながらない。ADR-0032 のリスク受容と合わせて別途）
- モノレポの名前、dx-portal を入れる時期、ports 台帳の生成方法（段 5 の最初に決める）
- self-hosted runner を置くホスト（候補: ステージングの `128.118.28.178`）とネットワークの条件
- IA 文書の所有者（`apps/adminnav/registry.py` を持つ人が自然）
- status の巡回を自作 cron にするか既製の監視にするか
