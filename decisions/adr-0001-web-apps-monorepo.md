# ADR-0001: Web アプリをモノレポに寄せる

**日付**: 2026-11-05
**ステータス**: 採用
**関連**: [ai-dev-standards ADR-0006](https://github.com/jazmf-dx/ai-dev-standards/blob/main/decisions/adr-0006-platform-composition-boundary.md)（Platform を統合入口とする。本 ADR で役割を変える）、[ADR-0002](adr-0002-pr-gates-runner-and-staging.md)、[ADR-0004](adr-0004-agent-working-agreement.md)

## コンテキスト

社内の Web アプリは 6 本（dx-portal・jazmf-directory・jazmf-tools・jazmf-knowledge・jazmf-lms・jazmf-workflow）が別リポジトリで、今後も増える。2026-11-05 の全体レビューで次が分かった。

- 共通コードを「写して」配っている。構造化ログは 6 本が同一ファイル、Directory 同期は 3 系統で約 5,900 行、死活監視と版の表示は 4 通り。dx-portal で直した誤り（X-Forwarded-For の取り方）が lms に残っている
- 共通資産の配り方が 2 度失敗している。ai-dev-platform の submodule は checkout で空のまま、`.claude/rules` の symlink は特定マシンの絶対パスを指して切れている
- エージェントのセッションで横断作業をすると 16 リポジトリを束ねることになり、そのとき各リポジトリの `.claude/settings.json`（SessionStart フック・許可）が読まれない。実際に 2026-11-05 のセッションで dx-portal のフックは走っていなかった
- 横断の判断（版の付け方・稼働中のコミット・リリース手順）が各アプリの ADR に 2〜4 重で書かれ、Standard 側の採用記録は全アプリが v1.0.0 のまま更新されていない

アプリが増えるほど「写す」「束ねる」のコストは増える。デスクトップアプリは既に desktop-apps で 1 リポジトリに複数ツールを置く形で動いている。

検討した選択肢:

| 案 | 内容 | 退けた理由 |
|---|---|---|
| 多リポジトリのまま共通パッケージと雛形を配る | GitHub Packages で `jazmf-django-common` を配り、新アプリは雛形から作る | パッケージの版上げを 6 か所で追う。横断変更が 6 PR。セッションを束ねる問題が残る |
| ポータルへ統合する | 衛星をやめて dx-portal に戻す | ポータル（18.8 万行）がさらに大きくなる。切り出しの判断（dx-portal ADR-0008）の逆行 |

## 決定

1. **Web アプリを 1 つのモノレポに寄せる。** 構成は `apps/<name>`（各アプリ）、`packages/`（共通: ログ・build info・status API・clientip・csvsafe・uploads・Directory クライアント・共通シェル）、`docs/`（原則・Standard・Playbook）、`decisions/`（横断の ADR）、`templates/`（新アプリの雛形）。
2. **デプロイはアプリごとのまま。** compose プロジェクトはアプリ単位で持ち、ホストの配置も当面変えない。モノレポは「コードと規約の置き場」であって、配布単位を 1 つにする決定ではない。
3. **段階的に移す。** 最初にモノレポを作り、共通パッケージと新しいアプリをそこに置く。既存の衛星は小さい順に `git subtree` で履歴ごと移す（workflow → tools → knowledge → lms → directory）。dx-portal は最後にするか、当面そのままにする。
4. **外に残すもの**: ui-platform（upstream の fork 運用のため）、desktop-apps と desktop-platform（デスクトップは別の Standard）、ports の台帳（モノレポの `docs/` に生成する形へ置き換えるまで）。
5. **ai-dev-platform の役割を変える。** 「読む文書の入口」から「雛形・共通パッケージ・reusable workflow・検査スクリプト・約束ブロックの配布元」にし、移行が済んだらモノレポの `docs/` と `packages/` と `templates/` に畳み込む。Standard の文書は 1 ページの原則集に圧縮し、Playbook は手順書として残す。
6. **セッションは 1 リポジトリで開く。** モノレポなら自然にそうなる。クラウドのセッションは sparse checkout で対象アプリに絞れる。

## 結果

- 共通コードの修正が 1 か所で済み、直した誤りが写し先に残らない
- CI・ルール・CLAUDE.md・ADR が 1 系統になる（[ADR-0002](adr-0002-pr-gates-runner-and-staging.md)・[ADR-0004](adr-0004-agent-working-agreement.md)）
- 新しいアプリを足すコストが「雛形から生成する」に揃う
- 移行の間は 2 つの置き場が並ぶ。その間、横断の決定は本リポジトリの `decisions/` を正とする
- CI は path filter と「変えたアプリだけ」の仕組みが前提になる（dx-portal の `scripts/app_deps.py` をモノレポ全体へ広げる）
- モノレポの名前、dx-portal を入れる時期、ports 台帳の生成方法は、モノレポを作るときに決める
