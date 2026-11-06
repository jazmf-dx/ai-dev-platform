# ADR-0006: 共通コードはモノレポの workspace で配り、UI Kit もモノレポに取り込む

**日付**: 2026-11-06
**ステータス**: 採用
**関連**: [ADR-0001](adr-0001-web-apps-monorepo.md)、[ADR-0003](adr-0003-ui-screen-types-and-shared-shell.md)、[ADR-0007](adr-0007-rules-live-in-monorepo.md)、ui-platform `decisions/adr-0002`（fork 運用と upstream への PR。本 ADR で改める）、dx-portal `decisions/project-context.md`「取得にトークンが要る」

## コンテキスト

- Python の共通コード（構造化ログ・build info・status API・clientip・csvsafe・uploads・Directory クライアント）はパッケージ化されておらず、ファイルを写して配っている。ログは 6 本が同一、Directory 同期は 3 系統・約 5,900 行。直した誤りが写し先に残る
- JS の共通は UI Kit（`@jazmf-dx/application-ui-kit`）で、ui-platform（個人側 hamirilo の upstream を jazmf-dx に fork したもの）が GitHub Packages に publish している。取る側は 6 か所（dx-portal・directory・tools・knowledge・lms と dx-portal の `pdf-one-web`）で、すべてに `read:packages` トークンが要る。トークンの渡し忘れが 2026-09-02 の本番デプロイ失敗（503 のまま）の原因になり、Claude Code on the web ではトークンが無いのでフロントの検査を飛ばしている
- 版が 7.0.0〜7.8.0 に散り、版上げを 6 か所で追っている

## 決定

1. **Python の共通コードはモノレポの `packages/` に置き、uv workspace のパス依存で使う。** GitHub Packages には出さない。publish もトークンも要らない。中身は ADR-0001 の一覧に加え、ジョブ基盤（ADR-0005）と共通シェル（ADR-0003）、OpenAPI から生成した Directory クライアント。
2. **ui-platform をモノレポの `packages/ui-kit/` に取り込み、bun workspace で参照する。** 取り込みは `git subtree` で履歴ごと。アプリの `package.json` は `application-ui-kit` の alias を workspace に向ける。Storybook はモノレポで動かす。
3. **upstream（hamirilo/ui-platform）との関係**: 取り込み後は jazmf-dx 側を正にする（[ADR-0007](adr-0007-rules-live-in-monorepo.md) と同じ判断）。upstream から取り込みたい変更があれば `subtree pull` で取る。upstream へ戻す運用（ui-platform ADR-0002 の「branch は upstream/main から切り、PR は upstream 宛」）はやめる。
4. **GitHub Packages への publish は、モノレポの外で使う人がいるときだけ。** いまの消費者は全員モノレポに入るので、移行が済んだら publish を止める。`@jazmf-dx/pdf-one-web`（デスクトップ側の pdf-one が出すもの。消費者は dx-portal だけ）は当面 GitHub Packages のままにし、トークンはモノレポの CI と環境の 1 か所に置く。`git+tag` で取る形への変更は pdf-one 側の判断に任せる。
5. **新しいアプリは雛形から生成する。** `templates/app/` + `just new-app <name>` で、共通パッケージ・UI Kit・worker・status・バックアップ・CI の path filter・CLAUDE.md の約束が入った状態で始まる。

## 結果

- 共通コードの修正が 1 か所で済み、全アプリに同時に効く。UI Kit の版は 1 つになる
- `read:packages` トークンが要る場所が 6 か所から 1 か所（pdf-one-web のため）に減る。Claude Code on the web でもフロントの検査が回るようになる
- ui-platform の fork 運用（upstream への PR）は終わる。個人側で ui-platform を使い続けるなら、モノレポから upstream への取り込みは手動で行う
- モノレポの clone に UI Kit と Storybook が含まれるので、クラウドのセッションは必要なら sparse checkout で絞る
- 移行の順: モノレポを作る → `packages/jazmf_django`（ログ・build info・status・clientip・csvsafe・uploads）→ ui-platform の subtree 取り込み → 衛星を移すたびに写しを消す → publish の停止
