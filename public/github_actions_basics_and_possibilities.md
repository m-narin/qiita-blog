---
title: 【GitHub Actions】基礎とその可能性
tags:
  - Git
  - GitHub
  - devops
  - CICD
  - GitHubActions
private: false
updated_at: '2026-05-25T11:44:26+09:00'
id: bb796b675c5a96ba04a3
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

社内LTで「GitHub Actions の基礎とその可能性」について発表した内容を、記事として一般化してまとめました。

CI/CD まわりは「すでに構築済みで触る機会が少ない」という方も多いと思いますが、改めて基礎を整理してみると、AI 全盛時代において改善のポテンシャルが大きい領域だと感じます。本記事は、GitHub Actions を「雰囲気で理解」している状態から一歩進んで、何ができるのかを言語化することを目的としています。

# なぜ今、GitHub Actions？

- AI 全盛時代において **CI/CD 改善のポテンシャル** は大きい
    - [動作確認の効率化（複数環境への同時デプロイ等）](https://qiita.com/MandoNarin/items/6dafdb96f1745aaefa6d)
- インパクトの大きい改善が生まれる領域の割に、注目されにくい
    - 開発現場では **すでに構築済み** のことが多く、触れる機会が少ないため
- 雰囲気で理解しがち → 「何ができるのか」を知ることが大事
    - 改善アイデアを想起するきっかけになる

---

# 1. 基本構文

ワークフローは **Workflow → Job → Step** の3階層で構成されます。

```yaml
name: CI                         # ワークフロー名

on: [push, pull_request]         # トリガー（いつ実行するか）

jobs:                            # ジョブの定義（並列実行の単位）
  test:
    runs-on: ubuntu-22.04        # どのマシンで実行するか
    steps:                       # ステップ（逐次実行の単位）
      - uses: actions/checkout@v4          # Action を使う
      - run: echo "Hello, World!"         # シェルコマンドを実行

  deploy:
    needs: test                  # test ジョブ成功後に実行
    runs-on: ubuntu-22.04
    steps:
      - run: echo "Deploying..."
```

| 概念 | 役割 | 実行 |
| --- | --- | --- |
| **Workflow** | 1つの YAML ファイル = 1つの自動化プロセス | トリガーで起動 |
| **Job** | 独立した実行単位。別のランナーで動く | デフォルト並列、`needs` で逐次 |
| **Step** | Job 内の各処理。`run`（コマンド）か `uses`（Action） | 上から順に逐次実行 |

## ディレクトリ構成

```
.github/
├── workflows/          # ワークフロー定義（YAML）
│   ├── backend-ci.yaml
│   ├── frontend-ci.yaml
│   ├── deploy-production.yaml
│   └── ...
├── actions/            # カスタムアクション（再利用可能な部品）
│   ├── setup-node-with-cache/action.yaml
│   ├── run-frontend-test/action.yaml
│   └── ...
└── scripts/            # シェルスクリプト
```

- **Workflow**: `.github/workflows/` 配下に YAML を置くと自動認識される
- **Action**: 処理の部品化。自作するか、[Marketplace](https://github.com/marketplace?type=actions) から利用する
    - 例: `actions/checkout@v4`、`actions/setup-node@v4`、`dorny/paths-filter@v3`
    - 自作した Action を公開することも可能（OSS 活動の機会にも）

---

# 2. 起動の仕方

多くの場合ワークフローは **イベント** によって起動しますが、起動パターンは多岐に渡ります。

```yaml
on:
  push:                          # プッシュ時
    branches: [master]
  pull_request:                  # PR 作成・更新時
    types: [opened, synchronize, reopened, labeled]
  schedule:                      # cron（定期実行）
    - cron: '0 0 * * 1-5'       # 平日 9 時（JST）
  workflow_dispatch:             # 手動実行（引数も取れる）
    inputs:
      environment:
        type: choice
        options: [staging, production]
  workflow_call:                 # 他ワークフローからの呼び出し（再利用）
```

新規ワークフロー作成時のテストについて

- 手動実行（`workflow_dispatch`）はブランチ指定で実行可能なため、テスト用ブランチで検証できる
- トリガー実行は対象ブランチに入れてから確認するしかなさそう

## トリガーの種類

公式ドキュメント: https://docs.github.com/ja/actions/reference/workflows-and-actions/events-that-trigger-workflows

- PR トリガーにした場合は、PR 上に UI が表示されるようになる

## 使い分けの例

| トリガー | 用途 |
| --- | --- |
| `pull_request` | CI（テスト・静的解析） |
| `push` (tag/branch) | デプロイ |
| `schedule` | カバレッジレポート、cron 監視 |
| `workflow_dispatch` | ラベルデプロイ等の手動操作 |
| `workflow_call` | ビルド・デプロイの共通化 |

---

# 3. ランナーとエフェメラル

## ランナー = ワークフローを実行するマシン

```yaml
jobs:
  test:
    runs-on: ubuntu-22.04
```

- **GitHub-hosted runner**: Ubuntu / macOS / Windows
- [インストール済みソフトウェア一覧](https://github.com/actions/runner-images) が公開されている（Docker, Node, Python, Ruby, etc.）
- **Self-hosted runner**: 自前マシンも利用可能

## エフェメラル（使い捨て）

- 毎回 **クリーンな環境** で実行される
    - セキュリティと再現性の担保
- 前回の状態は残らないため、**キャッシュ** や **Artifact** で状態を引き継ぐ

```yaml
# キャッシュの例: actions/setup-node の cache オプション
- uses: actions/setup-node@v4
  id: setup-node
  with:
    node-version-file: .node-version
    cache: pnpm # 次回以降キャッシュを利用できるようになる
```

```yaml
# Artifact の例: 実行結果を別 job に渡す / UI からダウンロード
- id: artifact-upload
  uses: actions/upload-artifact@v4
  if: ${{ !cancelled() }}
  with:
    name: playwright-report
    path: e2e/playwright-report/
    retention-days: 5
```

- UI 上からファイルをダウンロードしたり、別 job からファイルを参照したりできる

---

# 4. 課金モデル

- 無料枠を超えた分は従量課金
    - ストレージ利用料
    - 実行時間（マシンによって異なり、安いものだと約 $0.002 / 分）

参考: https://docs.github.com/ja/billing/concepts/product-billing/github-actions

---

# 5. コンテキストと環境変数

GitHub Actions には豊富な **コンテキスト** が用意されています。

例えば PR をトリガーに実行した場合、PR のコンテキストが参照可能になります。

```yaml
steps:
  - run: echo "${{ github.event.pull_request.head.ref }}"  # PR のブランチ名
  - run: echo "${{ github.sha }}"                          # コミット SHA
```

あるステップの結果を、後続のステップや別ジョブで参照することもできます（`$GITHUB_OUTPUT`）。

```yaml
jobs:
  update-deploy-branch:
    outputs:
      sha: ${{ steps.get-sha.outputs.sha }}
    steps:
      - name: Get current SHA
        id: get-sha
        run: echo "sha=$(git rev-parse HEAD)" >> $GITHUB_OUTPUT
      - run: echo "${{ steps.get-sha.outputs.sha }}" # step 間での参照

  build:
    needs: update-deploy-branch
    with:
      image_tag: staging-${{ needs.update-deploy-branch.outputs.sha }} # job 間
```

`sha=abc123def...` のようにキーバリュー形式で管理します。

- 同一 job の step 間で共有する場合、`steps.<id>.outputs.<key>` で直接参照
- job 間で共有する場合、`outputs` で公開し、`needs.<ジョブ名>.outputs.<key>` で参照
    - job が異なる = マシンが異なるので、明示的な受け渡しが必要

## GITHUB_TOKEN — 特別なクレデンシャル

- ワークフロー実行時に **自動発行** される一時トークン
- リポジトリへの読み書き、PR 操作、Issue 操作が可能
- 明示的に Secrets に登録する必要がない
    - 外部サービスを利用する場合、Settings > Secrets に登録して利用（例: `secrets.SLACK_WEBHOOK_URL`）

```yaml
- run: gh pr merge --auto
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

# 6. 並列実行と逐次実行

- Job はデフォルト並列。`needs` で逐次にできる
    - `needs` を設定した場合、GitHub Actions の UI 上で依存関係がグラフ表示される
- Step は常に逐次（上から順に実行）

```yaml
jobs:
  lint:   # job は並列実行
    steps:
      - run: bundle install   # step は逐次実行
      - run: bin/rubocop      # step は逐次実行
  test:   # job は並列実行
    steps:
      - run: bundle install
      - run: bin/rspec

  deploy:
    needs: [lint, test]   # lint と test が両方成功したら実行
    steps:
      - run: ./deploy.sh
```

## Matrix — 組み合わせの並列実行

```yaml
strategy:
  matrix:
    shard: ["1/2", "2/2"]    # テストを2分割して並列実行
  fail-fast: false           # 1つ失敗しても他は続行
```

```yaml
# 典型例: 複数バージョンでテスト
matrix:
  ruby: ["3.2", "3.3", "3.4"]
  os: [ubuntu-latest, macos-latest]
```

シャーディングと組み合わせれば、テストの実行時間短縮にも有効です。

---

# 7. 再利用パターン

## Reusable Workflow（ワークフローの共通化）

呼ばれる側は `workflow_call` をトリガーにします。

```yaml
# reusable-build.yaml
on:
  workflow_call:
    inputs:
      deployment_env:
        type: string
    secrets:
      NPM_TOKEN:
        required: true
```

呼ぶ側は `uses` でワークフローを指定します。

```yaml
# deploy-staging.yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yaml
    with:
      deployment_env: staging
    secrets: inherit
```

ビルドやデプロイ処理を Reusable Workflow にしておくと、staging / production / demo など複数環境で共通利用できます。

## Composite Action（ステップの部品化）

```yaml
# .github/actions/notify_job_result/action.yaml
name: Notify Job Result
description: Notify job result to Slack
inputs:
  deployment_env:
    required: true
  is_success:
    default: "true"
  channel:
    default: "#ci-notifications"

runs:
  using: composite
  steps:
    - run: |
        if [ "${{ inputs.is_success }}" == "true" ]; then
          bash ./.github/scripts/slack "Success: ..." "${{ inputs.channel }}" "good"
        else
          bash ./.github/scripts/slack "Failure: ..." "${{ inputs.channel }}" "danger"
        fi
      shell: bash
```

呼び出し側はこのように記述します。

```yaml
- uses: ./.github/actions/notify_job_result
  with:
    deployment_env: staging
    is_success: ${{ job.status == 'success' }}
```

## Action の命名規則とファイル構成

カスタムアクションは **ディレクトリ名 = アクション名** で、中に `action.yml`（または `action.yaml`）を配置します。

```
.github/actions/
├── setup-node-with-cache/     # ← アクション名（kebab-case が慣例）
│   └── action.yml             # ← 固定のファイル名（必須）
├── run-frontend-test/
│   └── action.yml
└── notify_job_result/
    └── action.yaml
```

公開リポジトリを参照する際にも、`action.yml` / `action.yaml` がエントリーポイントとして機能します（例: https://github.com/actions/checkout ）。ローカルの Action 参照でも同じルールが適用されます。

## 使い分け

|  | Reusable Workflow | Composite Action |
| --- | --- | --- |
| 再利用の単位 | **Job（ジョブ全体）** | **Step（ステップ群）** |
| 定義場所 | `.github/workflows/` | `.github/actions/` |
| 呼び出し方 | `jobs.X.uses:` | `steps[].uses:` |
| 用途 | ビルド・デプロイなど大きな処理 | セットアップ・通知など小さな処理 |

---

# 8. 実践テクニック

## パスフィルタリング — 変更がないジョブはスキップ

```yaml
- uses: dorny/paths-filter@v3
  id: changes
  with:
    filters: |
      backend:
        - 'app/**'
        - 'lib/**'
      frontend:
        - 'packages/**'

- if: steps.changes.outputs.backend == 'true'
  run: bin/rspec
```

モノレポ構成において、無駄な CI 実行を抑えるのに効果的です。

## concurrency — 古い実行を自動キャンセル

同じ `group` の古い Actions 実行を自動でキャンセルします。つまり、常に新しいものだけが実行されている状態を作れます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true # 同グループの実行中ジョブをキャンセル
```

- backend-ci のような CI: 古い実行は意味がないため
- ラベルデプロイのようなデプロイ系: 古い実行が後に完了すると不整合の原因になるため

## キャッシュ戦略

| 対象 | 方法 |
| --- | --- |
| Ruby gems | `ruby/setup-ruby` の bundler-cache |
| Node packages | `actions/setup-node` の pnpm/npm/yarn cache |
| システムパッケージ | `awalsh128/cache-apt-pkgs-action` |
| ESLint 結果 | `actions/cache` でカスタムキャッシュ |
| Docker イメージ | レジストリベースキャッシュ (`type=registry`) |
| テスト実行時間 | 実行時間ログを保存して分割を最適化 |

---

# 9. 活用事例とアイデア

## Dependabot の自動マージ

参考: [GitHub CI/CD実践ガイド](https://gihyo.jp/book/2024/978-4-297-14173-8)

パッチバージョンアップは自動 Approve & auto-merge する例です。

```yaml
- id: meta
  uses: dependabot/fetch-metadata@v2
- if: ${{ steps.meta.outputs.update-type == 'version-update:semver-patch' }}
  run: |
    gh pr review "${GITHUB_HEAD_REF}" --approve
    gh pr merge "${GITHUB_HEAD_REF}" --merge --auto
```

- Branch Protection と両立させるには、GitHub Actions による PR 承認を許可する設定が必要
- 全体適用が不安なら、自動マージ OK なライブラリをホワイトリスト管理する運用もあり

## AI によるコードレビューと自動ラベリング

参考: [GMO ペパボの事例](https://zenn.dev/pepabo/articles/ai-pr-review-bottleneck)

- PR に影響範囲のラベルを AI が自動付与
- 小さい変更はセルフマージ OK にする
- レビューのボトルネックを解消する

```yaml
name: Claude PR Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  claude-pr-review:
    runs-on: ubuntu-latest
    if: github.event.pull_request.draft == false
    steps:
      - uses: actions/checkout@v4

      # Claude Code Action でラベル付与 + レビューを実行
      - uses: anthropics/claude-code-action@v1
        with:
          direct_prompt: |
            以下の2つのタスクを順に実行してください。

            ## タスク1: PR サイズラベルの付与
            PR の変更内容を分析し、影響範囲に基づいて
            size/XS〜size/XL のラベルを付与してください。

            ## タスク2: コードレビュー
            PR をレビューし、問題がなければ Approve してください。

      # XS + Approve 済み → auto merge
      - name: Enable auto-merge for XS PRs
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          PR_NUMBER=${{ github.event.pull_request.number }}

          has_xs_label=$(gh pr view "$PR_NUMBER" \
            --json labels \
            --jq '[.labels[].name] | any(. == "size/XS")')

          is_approved=$(gh pr view "$PR_NUMBER" \
            --json reviews \
            --jq '[.reviews[] | select(.state == "APPROVED")] | length > 0')

          if [ "$has_xs_label" = "true" ] && [ "$is_approved" = "true" ]; then
            echo "size/XS かつ Approve 済み: auto-merge を有効化"
            gh pr merge "$PR_NUMBER" --auto --squash
          fi
```

レビュー基準やラベル分類基準は、リポジトリ内の `.claude/skills/` 配下に Markdown ファイルとして配置しておくと管理しやすいです。

## RSpec の失敗結果を Job Summary に表示

参考: https://zenn.dev/eggc/articles/20250319-github-actions-summary

`$GITHUB_STEP_SUMMARY` に書き込むだけで、Actions の Summary タブに結果を表示できます。

```yaml
- name: Run RSpec
  run: |
    set +e
    RESULT=$(bundle exec rspec)
    STATUS=$?
    set -e

    echo "$RESULT"

    # Finished in 以降（失敗サマリー部分）を Job Summary に出力
    echo '```' >> $GITHUB_STEP_SUMMARY
    echo "$RESULT" | awk '/^Finished in/ {p=1} p' >> $GITHUB_STEP_SUMMARY
    echo '```' >> $GITHUB_STEP_SUMMARY

    if [ $STATUS -ne 0 ]; then
      exit $STATUS
    fi
```

- failed examples のみを抽出するロジックを組めば、調査がさらに楽になる
- AI を組み合わせれば、対処方法も自動で提示できそう

## その他の活用アイデア

- 不要な CI 発火を抑える仕組み（ラベルでの CI 実行制御 など）
- AI による PR レビュー、Dependabot/Sentry の自動対応
- リリース対象の PR からリリースノートを自動生成
- Stale Issue/PR の自動クローズ（一定期間放置されたものを自動整理）
- 工数見積もり: AI による Issue への point 付与
- Issue 起票時に AI が類似 Issue を検索して重複を警告
- CI 失敗ログを AI が分析して原因と修正手順を PR コメントに投稿
- PR の差分から自動でテスト観点を生成し、QA チェックリストとしてコメント

## 注意点

- GitHub Actions 上で AI を使うと従量課金が発生する（Actions の課金 + AI API の課金）
- ローカルの Claude Max プラン等でできることは、そちらに流した方が節約になる（skills の活用など）

---

# おわりに

GitHub Actions は「すでに動いているもの」になりがちですが、その仕組みを理解すると、開発フロー全体を改善する大きなレバーになります。

特に AI との組み合わせは、レビューやテスト、Issue 整理など、これまで人手に頼ってきた部分を自動化できる可能性を秘めています。ぜひ自分のプロジェクトで「ここ自動化できそう」というポイントを探してみてください。

## もっと詳しく知りたい方へ

[GitHub CI/CD 実践ガイド](https://gihyo.jp/book/2024/978-4-297-14173-8) — 基礎から実践まで網羅的に解説されていておすすめです。
