# ADR-0003: UI は画面の型とナビを正にし、共通シェルを配る

**日付**: 2026-11-05
**ステータス**: 採用
**関連**: [ai-dev-standards ADR-0007](https://github.com/jazmf-dx/ai-dev-standards/blob/main/decisions/adr-0007-presentation-only-islands-and-template-classes.md)（テンプレート用クラスの契約）、ui-platform `templates/README.md`（「シェルは配らない」。本 ADR で改める）、dx-portal [ADR-0010](https://github.com/jazmf-dx/dx-portal/blob/main/decisions/adr-0010-frontend-stack-status-quo.md)（既存は移行せず新規だけ寄せる）、[ADR-0001](adr-0001-web-apps-monorepo.md)

## コンテキスト

デザインシステム（Semantic Token と `@jazmf-dx/application-ui-kit`）は UI の品質を保つために入れたが、「使いづらい」「縛られる」という実感があり、実測でも揃っていない。2026-11-05 の全体レビューより:

- 縛っているのは色トークンと部品（ボタン・バッジ）。一方で利用者が「ばらばら」と感じる**画面の骨格**（ページヘッダーと主ボタンの位置・一覧・フィルタ・空状態・タブ）と**ナビ**には基準が無い
- ui-platform はまさにその型（Standard App・一覧・詳細・集計のテンプレートと `tokens/classes.css` の約 100 クラス）を配っているが、dx-portal での使用は 0 件。`btn` `card` `badge` `data-table` を自前で定義し直している
- 結果、dx-portal ではボタン 5 系統（高さ 40 / 36 / 32px）、確認ダイアログ 6 系統、タブ 7 系統、h1 の書き方 64 通り、バッジの色 88 通り。新規の 56% が固定色で入り、2026-10-29 に作り直した管理コンソールのシェル自体が Alpine と固定色
- 規約の正本が 3 つ（DESIGN.md・`input.css`・ui-platform）で互いに矛盾し、人も AI も毎回どれに従うか判断している
- アプリ間ではシェルが 6 通り、ブランド名も別、戻り導線は tools にしか無い。Kit の `ProductSwitcher` は全アプリで未使用。利用者には 6 つの別アプリに見える
- DX通信のように独自の見た目が要る面は、規約と衝突して独自クラス（`mag-*` 306 個）を生んだ

## 決定

1. **縛る対象を「画面の型」と「ナビ」に移す。** 新しい画面は型を選ぶだけで、見た目の判断が要らない状態を目指す。
   - 一覧画面は page-header → filter-bar → data-table → pagination → empty の型
   - 詳細・集計・設定も ui-platform の型に従う
   - ページヘッダーの主ボタンは 1 つ、位置は型が決める
   - これらは ui-platform の `classes.css` と Template を取り込んで使う。同名のクラスを自前で定義し直さない
2. **共通シェルを配る。** ヘッダー・アプリ切り替え（ProductSwitcher）・通知・ポータルへの戻り導線・ユーザーメニューを、Django テンプレートの部品として共通パッケージ（[ADR-0001](adr-0001-web-apps-monorepo.md) の `packages/`）から配る。ui-platform の「シェルは配らない」方針は、React 部品としては維持し、Django テンプレートのシェルは jazmf-dx 側の共通パッケージが持つ。アプリは中身だけを書く。
3. **部品の正本は 1 つ。** React 側は Kit を正とし、各アプリの自前部品（dx-portal の `components/dx/`）は Kit に無いものだけ残す。DESIGN.md の部品定義は削除し、`input.css` と ui-platform を指す。確認ダイアログは confirm-host に一本化する。
4. **色の意味は 1 か所。** 状態の色はモデル側が tone（`badge-success` など）を返し、テンプレートは tone のクラスだけを書く。固定色の組み合わせを画面ごとに書かない。
5. **Focus App を例外として明記する。** DX通信の公開面・便利ツール・実験場のように独自の見た目が要る面は、トークンの上に載せる条件だけで独自の見た目を認める。規約違反として直さない。dx-portal の `mag-*` はその第 1 号として整理する。
6. **情報設計の基準を 1 枚持つ。** 管理画面のメニューはどのグループに置くか・タブかサブメニューか・`/manage/` の規則・「ダッシュボード」「一覧」「管理」の使い分け・社員向けナビに出す基準・二重実装の撤去期限を、ポータルの `docs/design/information-architecture.md` に 1 枚で決める。所有者は `apps/adminnav/registry.py` を持つ人。
7. **既存の扱いは dx-portal ADR-0010 のまま。** 一括置換はせず、新規・改修した画面から型に揃える。ただし「ついでに揃える」だけでは揃わなかったので、[ADR-0004](adr-0004-agent-working-agreement.md) の検査（固定色・Alpine・`<select>` の新規ゼロ）と組み合わせる。

## 結果

- 新しい画面の見た目の判断が減り、利用者はどのアプリでも同じ操作で済むようになる
- 共通シェルによって、ポータルと衛星が 1 つのシステムに見える。戻り導線と通知も共通になる
- ui-platform 側に Django テンプレート用の型の見本（Storybook の「テンプレート用クラス」）が正になるので、そこに無い型が要るときは先に ui-platform に足す
- Focus App の線引きは、揃えるべき面まで例外にしないよう、ADR か IA 文書で面ごとに明記する
- dx-portal での最初の作業は、`classes.css` の導入・管理画面の h1 の二重解消・`*_display_class` の tone 化・DESIGN.md の部品定義の削除。これらは新機能と並行できる
