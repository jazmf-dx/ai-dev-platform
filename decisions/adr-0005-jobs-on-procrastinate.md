# ADR-0005: バックグラウンド処理は Procrastinate のジョブに寄せ、ホストの crontab を無くす

**日付**: 2026-11-06
**ステータス**: 採用
**関連**: [ADR-0001](adr-0001-web-apps-monorepo.md)（共通パッケージの置き場）、[ADR-0002](adr-0002-pr-gates-runner-and-staging.md)、jazmf-directory `docs/design/01-architecture.md`（Celery + Redis を却下し Procrastinate を採用した判断）、dx-portal backlog WP-21（通知の outbox）・WP-24（定期ジョブ台帳）

## コンテキスト

「Go などの別言語や、ワーカー・キュー・別サービスが要るか」を 6 アプリで棚卸しした（2026-11-05）。

- Go / Rust のサーバーコードは無く、CPU が重い処理（PDF 編集・OCR・CSV 変換）はすでにブラウザ内の WASM と Web Worker に出してある。サーバー側で時間がかかるものの正体は外部 HTTP と subprocess の**待ち時間**（Garoon・Directory・fping・mtr・Claude API）で、言語を変えても速くならない
- 非同期の仕組みは jazmf-directory だけが持つ。**Procrastinate**（PostgreSQL をキューに使うジョブ基盤）+ worker コンテナで、Outbox の配送・定期の突き合わせ・再送を動かしている。Celery + Redis は「1,500 人規模ではブローカーの運用が見合わない」と却下済み
- 他のアプリは、メール・Teams・Garoon の配信を**リクエストの中で同期的に**行い（dx-portal・lms・workflow。lms の一括リマインドは N 通を直列に送りタイムアウトも無い）、出力や議事録の生成を gunicorn の中の `threading.Thread` で回している（再起動で「作成中」のまま残る）。大きなサブネットスキャンは gunicorn の 120 秒を超えうる
- 定期処理は全リポジトリで「ホストの crontab → just → docker compose exec」の形で、約 38 行を 4 ホストに手で登録している。登録漏れ・停止を検知する仕組みが無く、「一度も動いていなかった」前例がある
- キャッシュはどのアプリも未設定（プロセスごとの LocMemCache）で、レート制限がワーカー数ぶん緩む

dx-portal は「outbox テーブル + cron」で非同期化すると決めていたが（WP-21）、未実装だった。

## 決定

1. **ジョブ基盤は Procrastinate に統一する。** 共通パッケージ（ADR-0001 の `packages/`）に、アプリ定義・worker の起動・ジョブの記録（開始・成功・失敗・最終成功時刻）・再実行の口を入れ、各アプリは `@app.task` を書くだけにする。Redis や別のブローカーは入れない。
2. **各アプリの compose に worker サービスを足す。** web と同じイメージで `procrastinate worker` を動かす。fping など subprocess が要るジョブのために、worker のイメージにも同じツールを入れる。
3. **定期処理は Procrastinate の periodic task で持ち、ホストの crontab を無くす。** justfile の `*-setup` レシピは移行が済んだアプリから削除する。バックアップと logrotate のようにコンテナの外で動かすべきものだけ、ホストの cron に残す。
4. **最終成功時刻を `/api/status/` に出す。** ジョブごとの「期待する間隔」を超えて成功が無ければ `degraded` にする。これで WP-24（定期ジョブ台帳）は別の台帳を作らずに済む。
5. **リクエストの中から外へ出る処理はジョブに回す。** 対象は、メール・Teams・Garoon の配信（dx-portal・lms・workflow）、DX通信の承認と同時の配信、アンケートの出力、knowledge の議事録・現況・質問、「今すぐ計測」と大きなスキャン、Directory の「今すぐ取り込む」。画面は投入の結果を返し、進み具合はポーリングか HX-Trigger で知らせる。
6. **すぐ効く設定の手当て**（移行の前にやる）: lms と workflow に `EMAIL_TIMEOUT`、workflow の gunicorn に workers と threads、全アプリに `CONN_MAX_AGE`、レート制限とナビの件数のために共有キャッシュを **DatabaseCache** で設定する。Redis はチャットの即時化など ASGI / Channels が要るときに改めて決める。
7. **言語は変えない。** 想定外の要件（数千拠点の常時監視・数万同時接続の即時チャットなど）が出たときだけ、別サービスを改めて検討する。

## 結果

- 失敗が記録され、再実行でき、止まっていることが status で分かるようになる。cron の手登録と検知の問題が消える
- 画面が外部の応答を待たなくなり、gunicorn のワーカーが数十秒ふさがる状態が無くなる
- worker コンテナが 1 つ増える。ステージングと本番の compose、雛形（`just new-app`）に含める
- 移行の順: 共通パッケージに基盤を入れる → dx-portal の通知（WP-21 の置き換え）と定期処理 → lms・workflow・knowledge・tools → crontab の撤去。directory はすでに Procrastinate なので、基盤を共通パッケージに寄せるだけ
- dx-portal の WP-21（outbox + cron）は本 ADR で置き換える。backlog の該当行を更新する
