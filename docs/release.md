# リリース方針

この文書は、`ai-dev-platform`、`ai-dev-standards`、`ai-dev-playbook`、`ui-platform`のversionとreleaseの関係を定めます。各リポジトリ固有の実行方法は、そのリポジトリのREADMEまたはworkflowを正とします。

## 正式なrelease

- 各リポジトリの正式なreleaseは、SemVer形式のtag（`v<major>.<minor>.<patch>`）を伴うGitHub Releaseとする。
- npm package releaseは例外とし、`application-ui-kit-v<package-version>`形式のtagを使用する。
- tagだけを単独で作らず、変更点と必要な移行事項をRelease notesへ記載する。
- `main`は次のrelease候補であり、再現可能な採用versionとしてはrelease tagまたはcommitを指定する。
- すべてのmergeでreleaseする必要はない。関連する変更をまとめてreleaseしてよい。

## SemVerの判断

| 種別 | 判断 | 例 |
|---|---|---|
| PATCH | 既存方針と互換性を保つ修正・明確化 | 誤記修正、説明補足、手順の訂正 |
| MINOR | 既存利用者を壊さない追加 | 任意のStandard、後方互換なRecommendation / Playbook、新しい任意の選択肢の追加 |
| MAJOR | 採用側の判断や実装変更が必要な変更 | 既存方針の削除・反転、移行を要する必須Standardの追加・変更、共有資産の責務境界変更 |

迷う場合は、変更量ではなく**既存利用者が現在の採用方法を変える必要があるか**で判断します。

## Platformが固定する組合せ

`ai-dev-standards`と`ai-dev-playbook`は独立してreleaseできますが、個別のreleaseだけではPlatform利用者の組合せは変わりません。

`ai-dev-platform`のsubmodule pointerを更新してPlatformをreleaseした時点で、推奨するStandards / Playbookの組合せが確定します。Applicationは個別submoduleをlatestへ進めず、採用したPlatform tagまたはcommitが指す組合せを利用します。

## リリース順序

StandardsまたはPlaybookの変更をPlatformへ採用する場合は、次の順序で進めます。

1. 変更元リポジトリのPRをrelease候補として準備する。
2. PRのhead commitを基準に、関係するStandards / Playbook間の矛盾がないことをrelease前に確認する。
3. 確認済みのPRをmergeし、必要なversionで変更元リポジトリをreleaseする。
4. Platformでsubmodule pointerをrelease済みcommitへ更新する。
5. PlatformのRelease notesに、採用した各versionと主要な変更を記載する。
6. Platformをreleaseする。

StandardsとPlaybookの両方に変更がある場合は、原則としてStandards、Playbook、Platformの順にreleaseします。Playbookが新しいStandardを前提としない独立変更であれば、Standardsのreleaseは不要です。

## リポジトリごとの方法

| リポジトリ | release方法 | 備考 |
|---|---|---|
| `ai-dev-standards` | Release Please | release PRのversionと`CHANGELOG.md`を確認してmergeする |
| `ai-dev-playbook` | Release Please | release PRのversionと`CHANGELOG.md`を確認してmergeする |
| `ai-dev-platform` | Release Please | submoduleの組合せとRecommendationsをreleaseする |
| `ui-platform` | リポジトリとnpm packageを個別にrelease | 詳細は`ui-platform` READMEを正とする |

Standards / Playbook / Platformは同じ最小構成のRelease Pleaseを利用します。共通Workflow repositoryや追加設定は設けず、複数packageや独自tagを持つUI Platformは個別に管理します。fork判定は3 repositoryのWorkflowが同じ内容で持ちます。

具体的な導入・運用・失敗時の確認は[リポジトリのリリースPlaybook](../playbook/playbooks/repository-release.md)を参照してください。

## forkでの運用

共有資産をforkして利用する場合、forkは配布・検証・upstreamとの往復に使い、独自のreleaseは行いません（[ADR-0005](../standards/decisions/adr-0005-upstream-fork-operation.md)）。

- forkではRelease Pleaseを実行しない。3 repositoryのWorkflowがforkを判定して`release` jobをskipする。
- forkはupstreamのrelease tagを同期するだけで、独自のversionを切らない。`version.txt`と`CHANGELOG.md`はupstreamが所有する。
- forkがupstreamより先行する変更をApplicationが直ちに利用する場合は、fork上のcommitで固定する。upstreamがreleaseした後にtag固定へ戻す。
- Release notesの正本はupstreamのGitHub Releaseとする。forkにはtagがあれば足りる。

Platformのtag同期手順は[Adoption Guide](adoption.md)の「Platformを更新する」を参照してください。Standards / Playbookを含む一般的なfork運用とtag同期は[リポジトリのリリースPlaybook](../playbook/playbooks/repository-release.md#forkで利用する場合)を参照してください。

## UI Platform

`ui-platform`はPlatformのsubmoduleではなく、独立したrelease cycleを持ちます。

- `v<version>`: UI Platformリポジトリのrelease
- `application-ui-kit-v<package-version>`: npm packageのrelease

Applicationが利用するpackage versionのSource of Truthは、そのApplicationの`package.json`とlockfileです。Platform releaseへUI packageを同梱したり、Platformと同じversionへ揃えたりしません。

`ui-platform`をforkして利用する場合は、npm packageを利用組織のscopeでpublishするためpackage releaseはfork側でも行います。上記「forkでの運用」の対象はPlatform / Standards / Playbookであり、`ui-platform`のrelease方法は`ui-platform` READMEを正とします。

## Applicationのversion

この文書の対象は共有資産ですが、Applicationのversionの付け方もリポジトリの種類で分けて揃えます（dx-portal ADR-0048 決定5）。各Applicationの具体的な手順は、そのリポジトリのADRと`justfile`を正とします。

| 種類 | リポジトリ | version | release方法 |
|---|---|---|---|
| 利用者がversionを選ぶ共有資産 | `ai-dev-platform`・`ai-dev-standards`・`ai-dev-playbook`・`ui-platform`・`pdf-one-web` | SemVer | Release Please（上記） |
| 本番が1つのWeb Application | `dx-portal`・`jazmf-directory`・`jazmf-lms`・`jazmf-knowledge` | **release日**（`vYYYY.M.D`） | `just release` |
| 配布するDesktop Application | `stockroom`・`feedability`・`desktop-apps` | SemVer | tag方式 |

Desktop ApplicationがSemVerのままなのは、配布物のファイル名・アプリ内アップデート・利用者への表示にversionが出るためです。

### Web Applicationのversion（release日）

dx-portal ADR-0057 / jazmf-directory ADR-0014 で決めた形です（2026-10-30から）。`jazmf-lms`・`jazmf-knowledge`も同じ形にそろえる。

- versionは**releaseした日（日本時間）**とし、ゼロ埋めはしない。例: `v2026.11.3` = 2026年11月3日にreleaseしたもの
- 同じ日に2回目をreleaseするときは末尾に英字を付ける。2回目は`b`、3回目は`c`（`v2026.11.3b`・`v2026.11.3c`）。1回目には付けない（`a`は使わない）
- 並び順は`v2026.11.3` < `v2026.11.3b` < `v2026.11.3c` < `v2026.11.10`となる（`sort -V`と同じ）
- versionはPRで決めず、`just release`がreleaseするときに決める。tagとGitHub Releaseは本番に出たものにだけ付ける。GitHub Releaseの題には日付を日本語で添える（`v2026.11.3b（2026年11月3日・2回目）`）
- 英字が付くことがあるため、SemVer互換ではない。`GET /api/status/`の`version`は人が読む文字列として扱い、SemVerとして比較する道具に渡さない
- 切り替え前の`v2026.10.1`だけは「2026年10月の1回目」の意味で、日付ではない

### 稼働中のコミットの表示（Web Application共通）

「いま本番で動いているのはどのコミットか」を、**どのWeb Applicationでも同じ方法で確かめられる**ようにします。versionだけでは、同じ日のreleaseの違いや、releaseしていないmainからのデプロイを見分けられないためです。新しくWeb Applicationを作るときは、最初から次を満たします。

| 項目 | 内容 |
|---|---|
| 埋める値 | `APP_COMMIT`（`git rev-parse --short=7 HEAD`。追跡済みファイルに変更があれば末尾に`-dirty`。未追跡ファイルは無視する）と`APP_BUILT_AT`（UTCのISO8601）。**imageのbuild時**に埋める |
| 埋め方 | Dockerfileの`ARG GIT_COMMIT` / `ARG BUILT_AT` → `ENV APP_COMMIT` / `APP_BUILT_AT`。composeの`build.args`で`${GIT_COMMIT:-}` / `${BUILT_AT:-}`を渡し、`justfile`の先頭で`export GIT_COMMIT := ...`と`export BUILT_AT := ...`を評価する（buildする側のcheckoutの値が入る） |
| `GET /api/status/` | `version`（人が読む版。無ければ出さない）の次に任意フィールド`commit`・`built_at`。空なら**キーごと出さない**（嘘の値を出さない）。無認証なのでこれ以上は出さない |
| 全レスポンスのヘッダ | `X-App-Version`・`X-App-Commit`。`curl -I`で任意のページから確かめられる |
| 画面 | **アプリのトップページだけ**に、フッターの1行として小さく`v2026.11.3 · abc1234`（`title`に`built_at`。`-dirty`は琥珀色）。全ページには出さない（全員が同じ版を見るため）。メールには出さない |

- コンテナには`.git`が無いので、実行時に`git`を読む方式は採らない。必ずbuild時に埋める。
- 値が空（`just dev`の素のrunserverなど）のときは何も出さない。`unknown`のような偽の値で埋めない。
- 確認は`curl -s http://<ホスト>/api/status/`で`commit`を見て、`git log`と突き合わせる。`-dirty`が付いていたら、コミットされていない変更を載せてデプロイしている。
- 実装の実例: `dx-portal`・`jazmf-directory`・`jazmf-lms`・`jazmf-knowledge`の各`decisions/`にあるADR（「稼働中のコミットをstatus・ヘッダ・フッターに出す」）。
- Status API契約（`ai-dev-standards`の`standards/architecture/optional/status-api.md`）には`commit`・`built_at`を任意フィールドとして足す提案が要る（このリポジトリでは変更できないので、`ai-dev-standards`側で対応する）。

## Release notesの最小項目

PlatformのRelease notesには、少なくとも次を記載します。

- 採用した`ai-dev-standards` version
- 採用した`ai-dev-playbook` version
- Recommendationsの主な変更
- 既存Applicationで対応が必要な変更の有無

対応が不要な場合も「移行作業なし」と明記します。
