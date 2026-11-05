# AI Development Platform

開発時に「何を守るか・何を選ぶか・どう実施するか」へ適切にルーティングする **統合入口（composition root）** です。

Platformへすべての知識・実装を集約せず、責務ごとに分離した資産をApplicationから一貫した入口で利用できるようにします。

## 構成

```text
ai-dev-platform/
├── ai/
│   ├── ONBOARDING.md       AI agentが最初に読む唯一の入口
│   └── COMMITMENTS.md      全リポジトリの CLAUDE.md の先頭に置く「作業の約束」の正本
├── decisions/              jazmf-dx の組織横断の ADR（モノレポ移行までの置き場）
├── docs/
│   ├── adoption.md         Platformの導入・Applicationへの適用
│   ├── direction-2026-11.md 2026-11 に決めた方針と進め方
│   └── release.md          共有資産のversion・release方針
├── recommendations/        現時点の選択の正本
├── standards/              [submodule: ai-dev-standards]
└── playbook/               [submodule: ai-dev-playbook]
```

UIの具体的なdesign・実装資産は独立した [ui-platform](https://github.com/hamirilo/ui-platform) が所有します。

## 各領域の役割

| 領域 | 問い | 正本 |
|---|---|---|
| Standards | 何を守る？ | `standards/` |
| Recommendations | 普段は何を選ぶ？ | `recommendations/` |
| Playbook | どう実装・移行・検証・復旧する？ | `playbook/` |
| UI Platform | UIをどう設計・実装する？ | `ui-platform` |
| Project Context / ADR | このApplication固有ではどうする？ | 各Application |

同じ内容を複数箇所へcopyしません。Platformは **入口、routing、Recommendations、Standards / Playbookの組合せ** を担当します。

## 導入

Platformは複数のApplicationから参照できるworkspace上の位置へ1つ配置し、各Applicationの `CLAUDE.md` 等からPlatform側の `ai/ONBOARDING.md` を最初に読むよう参照させます。Application側には、実装へ大きく影響する前提を記録する `decisions/project-context.md` を用意します。

clone、submoduleの取得、参照の書き方、Project Contextの項目、既存Applicationへの段階適用は [Adoption Guide](docs/adoption.md) を参照してください。

## Workspace例

```text
workspace/
├── ai-dev-platform/
│   ├── standards/
│   ├── playbook/
│   └── recommendations/
├── ui-platform/
├── application-a/
└── application-b/
```

## 更新方法

Standards / Playbookのどの組合せを利用するかは、**Platformのsubmodule pointerを正**とします。Application側からStandards / Playbookを個別にlatestへ進めません。再現性が必要な場合はPlatform自体をrelease tagまたはcommitで固定し、そのcommitが指すsubmoduleを利用します。

更新手順とfork利用時のtag同期は [Adoption Guide](docs/adoption.md) の「Platformを更新する」を、SemVerの判断、release順序、forkでの運用は [リリース方針](docs/release.md) を参照してください。

## UI Platformとの関係

`ui-platform` はPlatformのsubmoduleにはしません。UIのrelease cycleとApplicationごとのpackage採用versionがStandards / Playbookとは異なるためです。

UI Platformが所有する範囲、`application-ui-kit` のpackage名とversionの扱いは、[ONBOARDING](ai/ONBOARDING.md) の「UI design / implementation」、[ADR-0006](standards/decisions/adr-0006-platform-composition-boundary.md)、[ADR-0005](standards/decisions/adr-0005-upstream-fork-operation.md) を参照してください。

## 知識・資産の成熟

```text
Project固有の判断・実装
        ↓
複数回利用して有効だった
        ↓
Recommendation / Playbook
        ↓
UIとして再利用するなら UI Platform
        ↓
不一致そのものが実害になる
        ↓
必要な場合のみ Standard
```

最初からすべてをStandardや共有資産へ昇格させません。

## メンテナンス

本repository自体のリンク検査用に最小限のtoolを用意しています。submoduleを初期化してから実行します。

```bash
just check-docs
```

## このrepositoryに置かないもの

- Standards本文のcopy
- Playbook本文のcopy
- UI Component / Pattern / Template / Storybook
- 業務domain固有情報
- Application固有の機能要件・運用手順
- 巨大なboilerplate
- AI agent定義、model設定、workflow実行基盤
