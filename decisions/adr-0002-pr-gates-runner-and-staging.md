# ADR-0002: PR ごとの検査・self-hosted runner・ステージングの自動化

**日付**: 2026-11-05
**ステータス**: 採用
**関連**: [ADR-0001](adr-0001-web-apps-monorepo.md)、dx-portal [ADR-0030](https://github.com/jazmf-dx/dx-portal/blob/main/decisions/adr-0030-ci-on-production-only.md)（CI は production だけ。本 ADR で置き換える）、dx-portal [ADR-0048](https://github.com/jazmf-dx/dx-portal/blob/main/decisions/adr-0048-verify-before-release-and-calver.md)・[ADR-0054](https://github.com/jazmf-dx/dx-portal/blob/main/decisions/adr-0054-smoke-test-and-light-release.md)（確かめてから出す・スモーク）、jazmf-directory ADR-0009

## コンテキスト

開発の主体がエージェントになり、PR は 1 日 8 本前後出る。ところが PR が main に入る前の機械的な検査が無い。

- dx-portal と jazmf-directory の CI は production 宛てにしか走らない（2026-09 末に org の Actions の分数を使い切った事故への対処）。main を守るのは手元の pre-push フックだけで、クラウドのセッションにはフックが入らず、フロントの検査は `CLAUDE_CODE_REMOTE` で飛ぶ
- 結果、失敗はリリース時の production CI で初めて見つかる。2026-11-04 には 2 件（テストの順番依存・フロントの後片付け）がそこで止まり、複数 PR 分を切り分けることになった
- CI の実測（dx-portal・2026-11-05）: 待ち時間 8〜9 分、ジョブの合計約 28 分。時間のほぼ全部が pytest（3 shard × 7.3 分。GitHub の標準ランナー 4 vCPU が頭打ち）。手元の高コア環境では同じテストが 145 秒
- ステージングは dx-portal だけにあり、載せるのも本番 DB を寄せるのも見るのも人の手。スモークテストは 2026-10-27 時点で「ステージングではまだ回していない」。衛星 5 本にはステージングが無い

「CI に時間がかかる」と「テスト環境の運用が追いつかない」は、どちらも人手の前提で組んだ仕組みがエージェントの速度に合っていないことが原因。

## 決定

検査を 3 層に分け、ステージングを自動化の対象にする。

| 層 | いつ | 何を | 目安 |
|---|---|---|---|
| **PR** | PR を開く・更新するたび | 静的検査（ruff・型・マイグレーション・依存の向き）+ **変えたアプリのテストだけ**（`app_deps.py affected`）。基盤（accounts・shared・organizations・notifications と共通パッケージ）を変えたときだけ全部 | 1〜3 分 |
| **main** | マージのたび | 全テスト + イメージビルド → **ステージングへ自動デプロイ → スモークを自動実行** | 5〜10 分。人は待たない |
| **リリース** | 火・金に人が押す | tag（CalVer）→ 本番デプロイ。既存の `just release deploy` の形 | 従来どおり |

1. **PR は必須にし、main への直接 push をやめる。** エージェントもローカルも同じ経路を通る。pre-push フックは補助に格下げし、無くても守られる状態にする。
2. **runner は self-hosted。** ステージングのホストに GitHub の self-hosted runner を置く。Actions の分数を消費しないので 2026-09 末の事故は起きない。同じ runner がステージングへのデプロイも行う（LAN 内なので SSH 越しの工夫が要らない）。必要なら 2 台目を本番ホストか待機系に置く。
3. **production ブランチを廃止する。** CI の節約のためだけに存在していた。リリースは main の tag だけで表す。dx-portal の release 系 ADR（0027 → 0047 → 0048 → 0057）の手順は「tag を打つ」に単純化する。
4. **ステージングは「常に main が動いている場所」にする。** 全アプリを 1 ホストに置き、main のマージで自動デプロイする。本番 DB の週次コピーは cron にする。人が見るのは、スモークが通ったあとの「今回変えた画面」だけ。
5. **`just check` は CI と同じ定義を呼ぶ 1 行にする。** 「CI と justfile の両方に足す」という人手の同期をやめる。クラウドのセッションで検査を飛ばす分岐も要らなくなる。
6. **justfile の役割は「デプロイと定期ジョブ」に絞る。** 検査・リリースの判断は CI が持つ。

## 結果

- 1 日 8 本の PR が main に入る前に検査され、失敗は PR の中で止まる
- ステージングに「載せる手間」が無くなり、確認は差分の画面だけになる。衛星にもステージングができる
- self-hosted runner のホストは社内の管理対象になる（OS 更新・ディスク・GitHub への outbound）。runner が止まると CI が止まるので、`/api/status/` と同じ巡回の対象に入れる
- `app_deps.py affected` の精度がゲートの精度になる。テンプレートの include と URL 名は import で拾えないので、基盤扱いにするか、テンプレートを変えたときの範囲を別に決める
- ADR-0030（CI は production だけ）と production ブランチの運用は、runner が動いた時点で終わる。dx-portal と jazmf-directory の ADR はステータスを「置き換え済み（本 ADR）」に更新する
- 着手の順: runner を置く → PR の workflow（affected）→ main の workflow（全部 + ステージング + スモーク）→ production ブランチの廃止。モノレポへの移行（ADR-0001）とは独立に、まず dx-portal で始めてよい
