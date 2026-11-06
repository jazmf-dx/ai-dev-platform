# ADR-0007: ルールはモノレポの docs/ に畳み、個人側 upstream への追従をやめる

**日付**: 2026-11-06
**ステータス**: 採用
**関連**: [ADR-0001](adr-0001-web-apps-monorepo.md)、[ADR-0004](adr-0004-agent-working-agreement.md)、[ADR-0006](adr-0006-shared-code-and-ui-kit-in-monorepo.md)、ai-dev-standards ADR-0003（Core / Optional）・ADR-0005（upstream fork 運用）・ADR-0006（Platform を統合入口とする）。本 ADR で ai-dev-standards ADR-0005・0006 の運用を終える

## コンテキスト

- 開発のルールは ai-dev-platform（入口・Recommendations）・ai-dev-standards（Standard）・ai-dev-playbook（手順）・ui-platform（設計文書）の 4 リポジトリに分かれ、文書は約 6,700 行（desktop-platform を含む。ui-platform の 643 行は別）。4 本とも個人側（hamirilo）の upstream を jazmf-dx に fork したもので、分かれている理由はこの fork 構造にある
- 配り方が 2 度失敗している。ai-dev-platform の submodule（standards・playbook）は checkout で空のまま、各アプリの `.claude/rules` は特定マシンの絶対パスへの symlink で切れている
- 各アプリの `standards-adoption.md` は全部 v1.0.0（2026-09-01）のまま更新されず、横断の判断（版の付け方・稼働中のコミット・リリース）はアプリの ADR に 2〜4 重で書かれている。ai-dev-standards の最終更新は 2026-09-10
- ルールの目的はトークン節約から「エージェントの出力を正しくする・機械で止める」に変わった（ADR-0004）。文書が多いほど矛盾が増え、読まれない

## 決定

1. **ルールの正本はモノレポの `docs/` にする。** 構成は `docs/principles.md`（原則 1 枚）、`docs/standards/`（Core だけを圧縮して残す。Optional は該当アプリの docs へ）、`docs/playbooks/`（手順書。いまの 13 本を必要なものだけ）、`docs/recommendations/`（ライブラリの既定）、`decisions/`（横断の ADR。本リポジトリの `decisions/` を移す）。
2. **個人側 upstream（hamirilo/*）への追従をやめる。** jazmf-dx が正。ai-dev-platform・ai-dev-standards・ai-dev-playbook は、モノレポへの移行が済んだ時点でアーカイブする（読み取り専用にし、README に移行先を書く）。
3. **配り方は「同じリポジトリの中」だけにする。** submodule・symlink・別リポジトリへのコピーは使わない。モノレポに無いもの（デスクトップ）だけ例外で、desktop-platform は desktop-apps の `docs/` に畳む（デスクトップは別モノレポ）。
4. **`standards-adoption.md` は廃止する。** モノレポにあること自体が採用の記録になる。逸脱は各アプリの `decisions/` の ADR に書く（Governance §1.1 の形式はそのまま）。
5. **文書は検査と対にする。** `docs/standards/` に残すルールは「どの検査が守っているか」を 1 行添える。検査が無いルールは原則集に 1 行で書くか捨てる（ADR-0004）。

## 結果

- ルールの置き場が 1 つになり、セッションを 1 つ開けば全部見える。矛盾の検出も 1 リポジトリの grep で済む
- 文書の総量を減らす作業が要る。目安は、原則 1 枚 + Standard 数百行 + 必要な Playbook。6,700 行をそのまま移さない
- 個人側（hamirilo）で同じ文書を使い続ける場合は、モノレポから個人側へ手動で持ち出す。自動の同期は作らない
- ai-dev-standards ADR-0005（fork 運用）・ADR-0006（Platform を統合入口に）は、アーカイブ時にステータスを「置き換え済み（本 ADR）」にする
- 移行の順: モノレポの `docs/principles.md` を書く → 本リポジトリの `decisions/` を移す → Standard を圧縮して移す → Playbook を選んで移す → 各アプリの CLAUDE.md の参照先を直す → 3 リポジトリをアーカイブ
